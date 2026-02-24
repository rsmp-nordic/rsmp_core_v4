# Proposed Changes: Status, Replay, Fetch & History

This document summarises the changes discussed for the RSMP 4 specification
and the Elixir reference implementation (`rsmp4`), with the reasoning behind
each one.

---

## Spec Changes

### 1. Rename "Full/Delta" to "Periodic/Event" ✅ done

**Files:** status.md, fetch.md

The old terms "Full Update" and "Delta Update" are misleading:

- "Delta" implies a diff against a known baseline, but RSMP deltas carry
  absolute values for a subset of keys — not mathematical differences.
- When a channel has only a single attribute, every "delta" is structurally
  identical to a "full" update, making the distinction confusing.

The new terms describe *why* the message was sent, not *how much data* it
contains:

- **Periodic Update** — triggered by a timer; always contains all attributes.
- **Event Update** — triggered by a state change; contains only changed
  attributes (plus Send Along).

### 2. Rename "Update rate" to "Periodic interval" ✅ done

**Files:** status.md

"Update rate" was ambiguous — it could refer to any kind of update. "Periodic
interval" makes it clear this setting controls the timer for periodic (all-
attribute) updates specifically.

### 3. Rename "Delta rate" to "Event rate" ✅ done

**Files:** status.md

Follows from the terminology change in item 1. "Event rate" describes how
event updates are triggered (on_change or at a fixed interval).

### 4. Add batching support ✅ done

**Files:** status.md, replay.md, history.md, channel.md

**Problem:** The original spec defines one event per MQTT message. For a
replay of 300 buffered items, this means 300 individual MQTT publishes.
On constrained cellular links, the per-message overhead (TCP/TLS/MQTT
headers) dominates the payload, wasting bandwidth and increasing broker
load.

**Solution:** Add an optional `batch_interval` channel configuration. When
set, the node accumulates events in memory and publishes them together as
an `entries` array at the configured interval.

Key design decisions:

- **Batching is optional.** Channels that need minimal latency (e.g. live
  signal groups for real-time display) omit the batch interval and publish
  immediately.
- **Batching is orthogonal.** Both periodic and event updates can be
  batched. Both replay and history responses can use the `entries` array.
- **Individual event precision is preserved.** Each entry in the array
  retains its exact `ts`, `seq`, and `values`. Unlike coalescing (which
  merges simultaneous changes into one event), batching preserves every
  intermediate state transition.
- **Replay and history benefit most.** When backfilling gaps, a node can
  send its entire buffer in a handful of large messages instead of hundreds
  of tiny ones, drastically reducing the "chattiness" of the protocol.

### 5. Add retention rules ✅ done

**Files:** status.md

**Problem:** The old spec simply said "full updates use retain=true, delta
updates use retain=false." This is correct for the common case but does not
cover edge cases (single-attribute channels, batched messages).

**Solution:** Define a single golden rule:

> A message MAY ONLY be published with `retain = true` if the payload
> represents a **complete data set** — it contains values for *all* primary
> attributes defined for that channel.

This rule naturally handles all cases:

- Periodic updates → always complete → retained.
- Event updates with multiple attributes → typically partial → not retained.
- Event updates for single-attribute channels → always complete → may be
  retained.
- Batched messages → retained only if the final entry is a complete data set.

### 6. Add `ts` field to status payloads ✅ done

**Files:** status.md

The original status payload only had `values` and `seq`. Without `ts`, the
consumer must use the MQTT arrival time as a proxy for when the event
occurred. This is unreliable — network latency, batching, and broker queuing
all distort arrival time.

Adding `ts` to every event ensures consumers can position data correctly in
a time series, consistent with the replay and history payloads which already
had `ts`.

### 7. Clarify coalescing vs. batching ✅ done

**Files:** status.md

The spec now clearly distinguishes two complementary mechanisms:

- **Coalescing** (`min_interval`): merges simultaneous changes into a single
  event object. If SG1, SG2, and SG3 all change within 100ms, one event is
  generated with all three values. Intermediate states are lost.
- **Batching** (`batch_interval`): groups sequential events into a single
  MQTT message. If SG1 changes at T=0 and SG2 changes at T=2, both events
  are preserved with their individual timestamps, but transmitted together
  at the batch boundary. No intermediate states are lost.

---

## App Changes (rsmp4 Elixir implementation)

### 8. Implement `truncated` flag

**Status:** ✅ done

The spec defines a `truncated: true` flag on replay and history messages
when the node's buffer did not cover the full requested range. The Elixir
implementation (`RSMP.Channel`) has a hardcoded `buffer_size: 300` but
never sets `truncated` when the buffer overflows.

Without this flag, the supervisor receives `complete: true` and assumes it
has all the data, silently leaving a permanent gap.

**Action:** When replay or history response starts at a point later than
the requested/expected start (because older data was evicted from the
buffer), set `truncated: true` on the final message.

### 9. Refactor replay to avoid `spawn` + `Process.sleep`

**Status:** ✅ done

`do_replay` in `channel.ex` spawns an unlinked process that loops through
the buffer with `Process.sleep` between publishes. This is an Elixir
anti-pattern:

- The spawned process is not linked or monitored — if it crashes, nobody
  knows.
- `Process.sleep` ties up a BEAM scheduler thread just to wait.
- The spawned process cannot be cancelled if the connection drops.

The history sending already uses the correct pattern: state is tracked inside
the GenServer (`active_send`), and `Process.send_after` is used for rate
limiting.

**Action:** Refactor replay to use the same in-GenServer pattern as history:
track replay state in the Channel GenServer state, advance via
`Process.send_after`, and cancel cleanly on disconnect.

### 10. Track replay progress for interrupted replays

**Status:** ✅ done

The spec mandates: "On the next reconnect the node resumes replay from where
it left off — it MUST NOT re-send messages it has already replayed."

The current implementation does not track where a replay was interrupted. On
every reconnect, it replays from whatever `since` timestamp is provided,
potentially re-sending data.

**Action:** Store the last successfully replayed `seq` (or `ts`) in the
Channel state. On reconnect, resume from that point instead of from the
beginning.

### 11. Implement batching in the app

**Status:** ✅ done

Once the spec defines batching, the Elixir implementation should support it:

- Add `batch_interval` to channel configuration.
- In `RSMP.Channel`, accumulate events in a `current_batch` list.
- On timer expiry, publish the batch as `%{"entries" => [...]}`.
- For history/replay responses, send entries in chunks rather than one at a
  time, eliminating the `send_next_history_entry` loop for large responses.

### 12. Improve data point eviction performance

**Status:** ✅ done

In `RSMP.Remote.Node.Site.store_data_point`, when the map reaches
`@max_data_points` (3600), the oldest entry is evicted using `Enum.min_by`
over the entire map — an O(N) operation on every insert when the buffer is
full.

**Action:** Use a more efficient data structure for the time-series sliding
window. Options include:

- A `{:queue, map}` pair where the queue tracks insertion order for O(1)
  eviction.
- An ETS ordered_set keyed by timestamp for O(log N) operations.
- A simple list with periodic compaction.
