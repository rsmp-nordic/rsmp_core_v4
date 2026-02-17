# RSMP 4 Stream Concept — Analysis and Recommendations

## 1. Executive Summary

The stream concept in RSMP 4 aims to replace the per-attribute subscription model of RSMP 3 with a pre-configured, topic-based publishing model that fits naturally with MQTT. This analysis evaluates the current draft against real-world TLC use cases, MQTT best practices, and known RSMP 3 pain points — particularly around S0001 signal group status and cycle counter precision.

The concept is fundamentally sound. The main areas needing work are:

- Clarifying how streams map to MQTT topics, retained messages, and QoS
- Tightening the "Send on Change" vs "Send Along" attribute distinction to solve the S0001/cyclecounter problem
- Defining delta encoding with enough precision for implementers
- Adding rate-limiting and throttling semantics
- Specifying stream listing and administration as concrete MQTT interactions
- Addressing per-component vs bundled publishing for high-frequency statuses

---

## 2. Context: RSMP 3 Status Subscription Pain Points

### 2.1 The S0001 Problem

S0001 bundles four attributes into one status:

| Attribute | Change frequency | Nature |
|---|---|---|
| `signalgroupstatus` | Every signal transition (every ~10–30 s, sometimes sub-second) | Primary data |
| `cyclecounter` | Every 1 s (or 100 ms with higher precision) | Metadata / time-reference |
| `basecyclecounter` | Every 1 s (or 100 ms) | Metadata / time-reference |
| `stage` | Every stage change (~10–30 s) | Primary data |

In RSMP 3 with `sendOnChange`, the cyclecounter triggers a status message **every second** (or every 100 ms if precision is increased), even when signal groups haven't changed. For a typical intersection with no signal changes, this produces ~60 needless messages per minute, or ~600/min at 100 ms resolution.

This was discussed extensively in issue [rsmp_sxl_traffic_lights#21](https://github.com/rsmp-nordic/rsmp_sxl_traffic_lights/issues/21) and the "lean attributes" concept in [rsmp_core#189](https://github.com/rsmp-nordic/rsmp_core/issues/189). The lean attributes idea was ultimately abandoned for RSMP 3 and deferred to RSMP 4.

### 2.2 RSMP 3 Subscription Complexity

In RSMP 3, the subscriber must specify `updateRate` and `sendOnChange` per attribute at subscription time, sent over a point-to-point TCP connection. This creates several problems:

1. **Per-connection configuration**: Each supervisor must negotiate its own subscription settings.
2. **No shared benefit**: If 5 supervisors want the same data, the site publishes it 5 times.
3. **Tight coupling**: The subscriber must understand the attribute semantics to set good subscription parameters.
4. **No broker buffering**: If the connection drops, data is lost.

The stream concept addresses all of these by moving subscription configuration to the publisher side, leveraging MQTT's pub/sub fan-out.

---

## 3. Evaluation of Current Stream Draft

### 3.1 What Works Well

**Pre-configured publishing.** Making streams pre-defined rather than negotiated at subscription time is the right architectural choice for MQTT. Multiple consumers benefit from the same published data without additional load on the device.

**Multiple streams per status type.** The "live" vs "hourly" example in the draft is a strong pattern. It lets a single device serve both real-time monitoring and periodic reporting use cases simultaneously.

**Default on/off state.** Sensible for bandwidth management. High-frequency streams like S0001 live updates should default to off, while infrequent ones like S0014 (current time plan) can default to on.

**Pruning / auto-stop.** Important for preventing abandoned high-bandwidth streams from running indefinitely. Tying this to the presence mechanism is a natural MQTT fit.

**"Send on Change" vs "Send Along" attribute types.** This is exactly the mechanism needed to solve the S0001/cyclecounter problem. The cyclecounter should be "Send Along" — included when the signalgroupstatus triggers an update, but not triggering updates on its own.

### 3.2 What Needs Clarification or Improvement

The following sections detail specific issues and recommendations.

---

## 4. Mapping Streams to MQTT Primitives

### 4.1 Issue: Topic Structure for Streams

The draft shows streams as named entities (e.g., `traffic/live`, `traffic/hourly`) but doesn't define how these map to the RSMP 4 topic structure: `<node>/<type>/<code>[/<component>]`.

**Recommendation:** Streams should publish to standard RSMP 4 status topics, with the stream variant distinguished by a suffix or subtopic:

```
dk/cph/45fe/status/tlc.signalgroup/sg/1           # default / full-rate stream
dk/cph/45fe/status/tlc.signalgroup.hourly/sg/1     # hourly aggregated stream
```

Or alternatively, use a `stream` level:

```
dk/cph/45fe/stream/live/tlc.signalgroup/sg/1
dk/cph/45fe/stream/hourly/tlc.signalgroup/sg/1
```

The second approach has the advantage that you can subscribe to `dk/cph/45fe/stream/live/#` to get all live streams, or `dk/cph/45fe/stream/#` for everything.

Consider using MQTT topic aliases (MQTT 5) for frequently published high-volume topics to reduce per-message overhead.

### 4.2 Issue: Retained Messages and Full vs Delta Updates

The draft mentions "full updates, which are retained" and "delta updates which are not retained." This is the correct approach but needs to be stated as a firm rule.

**Recommendation:** Make this an explicit protocol requirement:

- **Full updates** MUST be published with MQTT `retain = true`. They represent the complete current state. A new subscriber will immediately receive the latest full state.
- **Delta updates** MUST be published with MQTT `retain = false`. They represent changes since the last full or delta update. A new subscriber should NOT receive a stale delta.

This maps cleanly to how MQTT retained messages work. When a subscriber connects and subscribes to a stream topic, they get the latest retained full update, then receive deltas as they occur.

### 4.3 Issue: QoS Level

Not mentioned in the draft. Different streams have different reliability needs.

**Recommendation:** Define default QoS per stream, with the option to override:

| Stream type | Suggested QoS | Rationale |
|---|---|---|
| High-frequency live data (S0001) | QoS 0 | Losing one update is acceptable; the next one arrives within seconds. Avoids overhead of acknowledgement. |
| Periodic aggregated data (traffic counting) | QoS 1 | Each message represents a time window worth of data; loss is costly. |
| Critical state changes (alarms, S0005 startup) | QoS 1 | Must be delivered reliably. |
| Configuration data (S0098) | QoS 1 | Infrequent, high-value data. |

SXLs should specify the recommended QoS for each stream definition.

---

## 5. Solving the S0001 / Cyclecounter Problem

### 5.1 "Send on Change" vs "Send Along" — Make It Central

The draft defines these as attribute types but doesn't elaborate on the semantics. This is the critical mechanism that solves the S0001 cyclecounter problem and should be promoted to a first-class concept.

**Recommendation:** Define clearly:

- **Send on Change (primary):** This attribute's value change triggers publication of a delta update. The update includes the new value of this attribute plus the current values of all "Send Along" attributes.
- **Send Along (secondary):** This attribute is included in every update, but its value changing alone does NOT trigger publication. It rides along when a primary attribute triggers.

**Applied to S0001:**

| Attribute | Type | Behavior |
|---|---|---|
| `signalgroupstatus` | Send on Change | Triggers publication when any signal group changes state |
| `stage` | Send on Change | Triggers publication when stage changes |
| `cyclecounter` | Send Along | Included in every message but doesn't trigger one |
| `basecyclecounter` | Send Along | Included in every message but doesn't trigger one |

**Result:** During a 30-second period with no signal transitions, zero messages are sent — instead of 30 (or 300 at 100 ms resolution). When a signal group does change, the message includes the precise cyclecounter at the moment of change. This is actually *more useful* than getting a cyclecounter update every second, because the cyclecounter value is tied to the exact moment of the state transition.

### 5.2 Edge Case: Cyclecounter Still Needed Periodically

Some consumers may want periodic cyclecounter updates for coordination monitoring, even when signal groups aren't changing.

**Solution:** This is exactly what the **Update Rate** setting handles. A stream can be configured with:

- **Delta Rate:** Send on change (for signalgroupstatus transitions)
- **Update Rate:** e.g. 5 seconds (periodic full update including cyclecounter)

This way a consumer gets immediate deltas on signal changes PLUS a full state snapshot every 5 seconds for coordination monitoring. The full update is retained on the broker, so new subscribers get the latest state.

### 5.3 Higher Precision Cyclecounter

Issue #21 discussed adding a `cyclecounter10` attribute with 100 ms resolution. With the Send Along mechanism, this becomes safe to add — it won't trigger 10 updates/second because it's a Send Along attribute. It will simply provide higher precision timestamps on actual signal transitions.

---

## 6. Delta Encoding

### 6.1 Issue: Delta Format Not Defined

The draft mentions "delta updates contain only attributes that changed" but doesn't define the encoding.

**Recommendation:** Define a clear delta format for each data type:

**For structured objects (most statuses):**
Deltas include only the attributes that changed (sparse update). The receiver merges the delta into its last known full state.

```cbor
# Full update (retained):
{"signalgroupstatus": "BBBA1NGBBBB", "cyclecounter": 45, "basecyclecounter": 45, "stage": 3}

# Delta (not retained) — only signalgroupstatus and stage changed:
{"signalgroupstatus": "BBBA2OGBBBB", "stage": 4, "cyclecounter": 52, "basecyclecounter": 52}
```

Note: Even in delta messages, Send Along attributes are always included, since their role is to provide context for the change.

**For string-encoded arrays (S0001 signalgroupstatus, S0002 detectorlogicstatus):**

Consider whether character-level deltas are worth the complexity. For S0001, the signalgroupstatus string is typically 5–50 characters. At these sizes, sending the full string is simpler and the bandwidth saving of per-character deltas is negligible.

**Recommendation:** Keep deltas at the attribute level, not within attribute values. This keeps the model simple and avoids a new sub-encoding.

### 6.2 Per-Component Publishing vs Bundled Strings

In RSMP 3, S0001 sends a single signalgroupstatus string encoding ALL signal groups as characters. In RSMP 4, with per-component topics, there's an architectural choice:

**Option A: One topic per signal group**
```
dk/cph/45fe/status/tlc.signalgroup/sg/1  → {"state": "1", "cyclecounter": 45}
dk/cph/45fe/status/tlc.signalgroup/sg/2  → {"state": "B", "cyclecounter": 45}
```

**Option B: Bundled into one topic**
```
dk/cph/45fe/status/tlc.signalgroup  → {"signalgroupstatus": "1B...", "cyclecounter": 45}
```

**Trade-offs:**

| | Option A (per-component) | Option B (bundled) |
|---|---|---|
| MQTT retained messages | Each signal group's state retained independently | Single retained message for all |
| Granular subscription | Subscribe to one signal group | All or nothing |
| Message count | Many small messages per cycle | One message per cycle |
| Bandwidth | More overhead from topic repetition and MQTT headers | More compact |
| Atomicity | No guarantee all signal groups arrive together | Atomic snapshot |
| Broker load | More retained messages to manage | Fewer |

**Recommendation:** Use **Option B (bundled)** for S0001-type statuses that represent a synchronized snapshot. Atomicity matters for signal group diagrams — you need the state of ALL groups at the same cyclecounter. Per-component publishing would make it hard to reconstruct a consistent snapshot.

However, offer Option A for statuses where per-component independence is natural (e.g. S0025 Time-of-Green/Red per signal group, alarm per detector logic).

This is a decision for SXLs to make per status type, not a core protocol choice. The stream concept should support both patterns.

---

## 7. Aggregation

### 7.1 Current Concept is Sound

The aggregation options (count, average, median, max, min) are well-suited for traffic counting statuses (S0201–S0208). For example:

- S0201 (vehicle count) → aggregation: **sum** over 15-minute windows
- S0202 (vehicle speed) → aggregation: **average** over 15-minute windows
- S0203 (occupancy) → aggregation: **average** or **max**

### 7.2 Clarifications Needed

**Aggregation window alignment:** Should aggregation windows be aligned to clock boundaries (e.g. every 15 minutes on the quarter-hour) or relative to stream start? Clock-aligned windows are standard practice in traffic engineering and recommended.

**Partial windows:** What happens when a stream starts mid-window? Options:
1. Discard the partial window (clean but loses data)
2. Send a partial window with a flag (more complete)

**Recommendation:** Support both. Include a `partial` flag in the message when the aggregation window was incomplete.

**Aggregation + Send on Change:** Can these be combined? For instance, "send the 15-minute vehicle count, but only if it's non-zero." This could reduce unnecessary messages from detectors with no traffic.

---

## 8. Rate Limiting and Throttling

### 8.1 Issue: Not Addressed in Draft

The draft doesn't discuss rate limiting. For high-frequency data like S0001 with rapid signal changes, a consumer may not need or be able to handle 10 updates/second.

**Recommendation:** Add a **minimum interval** setting to streams:

- **Minimum interval:** The shortest time between consecutive delta publications. Changes occurring within this interval are coalesced into the next publication.

Example: S0001 live stream with `minimum_interval: 100ms`. If three signal group changes occur within 100 ms, a single delta is sent containing all three changes.

This gives device implementers a way to bound the maximum message rate while still being responsive.

### 8.2 Maximum Message Size

For statuses with potentially large payloads (S0098 configuration dump, S0204 vehicle classification across all detectors), consider defining a maximum recommended message size and a chunking mechanism if needed.

For MQTT, messages up to ~256 MB are theoretically supported, but practical limits are much lower. A reasonable guideline might be 64 KB per message.

---

## 9. Stream Lifecycle and State Transitions

### 9.1 Issue: Start/Stop Semantics Need Specification

The draft says "when you start a stream, it starts publishing data to broker." But it doesn't define the mechanism for starting/stopping.

**Recommendation:** Define stream control as RSMP 4 commands:

```
dk/cph/45fe/command/stream.start    → {"stream": "tlc.signalgroup.live"}
dk/cph/45fe/command/stream.stop     → {"stream": "tlc.signalgroup.live"}
```

Or, since streams exist at the topic level, you could use a dedicated stream-control topic:

```
dk/cph/45fe/stream-control   → {"action": "start", "stream": "tlc.signalgroup.live"}
```

The response should confirm the action and the resulting stream state.

### 9.2 Stream State Machine

Define explicit states:

```
stopped → starting → running → stopping → stopped
```

- **stopped:** No data published. No retained message on broker (or a retained message indicating "stream stopped").
- **starting:** Device is initializing the stream (e.g. loading configuration, starting data collection).
- **running:** Data is being published according to stream configuration.
- **stopping:** Device is flushing final data and cleaning up.

When a stream is stopped, the device SHOULD publish a retained message with empty/null payload to clear the retained state on the broker, so new subscribers don't receive stale data. (MQTT allows clearing retained messages by publishing an empty payload with retain=true.)

---

## 10. Stream Discovery and Listing

### 10.1 Issue: "Listing Available Streams" Not Defined

The draft mentions "the node provides a list of available streams" but doesn't specify how.

**Recommendation:** Publish the stream catalog as a retained status message:

```
dk/cph/45fe/status/stream.catalog → {
  "streams": [
    {
      "id": "tlc.signalgroup.live",
      "code": "tlc.signalgroup",
      "attributes": {
        "signalgroupstatus": {"type": "on_change"},
        "stage": {"type": "on_change"},
        "cyclecounter": {"type": "send_along"},
        "basecyclecounter": {"type": "send_along"}
      },
      "update_rate": 5000,
      "delta_rate": "on_change",
      "min_interval": 100,
      "default_state": "off",
      "qos": 0,
      "aggregation": null,
      "prune_timeout": 300
    },
    {
      "id": "tlc.traffic.15min",
      "code": "tlc.traffic",
      "attributes": {
        "vehicles": {"type": "on_change", "aggregation": "sum"},
        "speed": {"type": "on_change", "aggregation": "average"},
        "occupancy": {"type": "on_change", "aggregation": "average"}
      },
      "update_rate": 900000,
      "delta_rate": null,
      "default_state": "on",
      "qos": 1,
      "aggregation": "window",
      "prune_timeout": null
    }
  ]
}
```

This gives consumers a machine-readable way to discover what streams are available, their configurations, and semantics.

---

## 11. Stream Administration

### 11.1 Realistic Scope

The draft mentions adding, editing, and removing streams remotely. This is powerful but raises complexity:

- **Validation:** Not all attribute/rate/aggregation combinations are meaningful. Devices must validate requests.
- **Persistence:** Are custom streams persistent across device reboots?
- **Limits:** Devices have finite CPU and memory. There must be a way to reject stream creation.

**Recommendation:** Define stream administration as commands with explicit error responses:

```
dk/cph/45fe/command/stream.add → {
  "id": "custom.signalgroup.fast",
  "code": "tlc.signalgroup",
  "attributes": { ... },
  "update_rate": 1000,
  "delta_rate": "on_change",
  "min_interval": 50
}
```

Result:
```
dk/cph/45fe/result/stream.add → {
  "status": "ok",
  "id": "custom.signalgroup.fast"
}
```

Or on failure:
```
dk/cph/45fe/result/stream.add → {
  "status": "error",
  "reason": "min_interval below device minimum (100ms)"
}
```

### 11.2 Access Control

The draft correctly notes that MQTT broker ACLs can manage access to stream administration. This should be the primary access control mechanism. Devices should also be able to reject requests that exceed their capabilities.

---

## 12. Pruning Details

### 12.1 Mechanism

The draft mentions using presence messages and timeouts. Concrete recommendation:

1. **Subscriber-count based:** The device monitors MQTT subscription counts (if the broker supports `$SYS` topics or MQTT 5 subscription identifiers). When the count drops to zero for a stream's topic, start a timer. If no new subscribers appear within the prune timeout, stop the stream.

2. **Presence-based:** The device watches presence topics.  When all known consumers go offline, start the prune timer.

3. **Explicit keep-alive:** Consumers periodically publish to a keep-alive topic. If no keep-alive is received within the timeout, prune.

**Recommendation:** Method 3 (explicit keep-alive) is the most reliable and portable across MQTT broker implementations. Method 2 (presence-based) is simpler and may be sufficient for most cases.

### 12.2 Prune Behavior

When pruning triggers, the device should:
1. Stop the stream (stop publishing)
2. Clear the retained message on the broker (publish empty payload with retain)
3. Update stream state in the catalog

---

## 13. Categorization of TLC Statuses by Stream Pattern

To validate that the stream concept covers real use cases, here is how TLC statuses from the SXL map to stream patterns:

### 13.1 High-Frequency Live Data

| Status | Description | Recommended Stream Pattern |
|---|---|---|
| S0001 | Signal group status | Delta on change + full update every 5 s. Send Along: cyclecounter, basecyclecounter. Default: off. |
| S0002 | Detector logic status | Delta on change + full update every 5 s. Default: off. |
| S0003 | Input status | Delta on change + full update every 10 s. Default: off. |
| S0004 | Output status | Delta on change + full update every 10 s. Default: off. |
| S0025 | Time-of-Green / Time-of-Red | Per signal group, delta on change. Default: off. |
| S0033 | Signal Priority Status | Delta on change. Default: off. |

### 13.2 Low-Frequency State Changes

| Status | Description | Recommended Stream Pattern |
|---|---|---|
| S0005 | Controller starting | Delta on change. Default: on. Retained. |
| S0007 | Controller switched on | Delta on change. Default: on. Retained. |
| S0008–S0013 | Manual/fixed/isolated/yellow flash/all red/police key | Delta on change. Default: on. Retained. |
| S0014 | Current time plan | Delta on change. Default: on. Retained. |
| S0015 | Current traffic situation | Delta on change. Default: on. Retained. |
| S0020 | Control mode | Delta on change. Default: on. Retained. |
| S0032 | Coordinated control | Delta on change. Default: on. Retained. |

These change infrequently (minutes to hours). A simple "delta on change with retained full update" stream is appropriate. No aggregation needed.

### 13.3 Periodic / Aggregated Data

| Status | Description | Recommended Stream Pattern |
|---|---|---|
| S0201–S0204 | Traffic counting (per-detector) | Aggregated over configurable windows (1 min, 5 min, 15 min). QoS 1. |
| S0205–S0208 | Traffic counting (all detectors) | Aggregated over configurable windows. QoS 1. |

### 13.4 On-Demand / Configuration Data

| Status | Description | Recommended Stream Pattern |
|---|---|---|
| S0016–S0017, S0019 | Number of detectors/signal groups/etc. | No stream. Request-response (use command/result). |
| S0022–S0028 | Time plans, tables, offsets, cycle times | No stream. Request-response or very low-rate full update. |
| S0095 | Controller version | No stream. Request-response. |
| S0096 | Current date/time | No stream. Request-response. |
| S0097–S0098 | Config checksum/download | No stream. Request-response. |

Some statuses are inherently request-response rather than streaming. The stream concept should not try to cover these — they belong in the command/result pattern. The SXL should clearly identify which statuses are suitable for streaming and which are request-response only.

---

## 14. MQTT 5 Features to Leverage

RSMP 4 specifies MQTT 4, but should consider MQTT 5 if feasible. Relevant features:

| Feature | Benefit for Streams |
|---|---|
| **Message Expiry Interval** | Full updates can expire after 2× the update rate. Prevents stale retained messages if a device goes offline without cleaning up. |
| **Topic Aliases** | Reduces per-message overhead for high-frequency topics like S0001. |
| **Shared Subscriptions** | Allows load-balanced consumption of high-volume streams across multiple backend instances. |
| **User Properties** | Can carry stream metadata (stream ID, sequence number) without encoding it in the payload. |
| **Subscription Identifier** | Helps consumers correlate incoming messages with their subscriptions. |

**Recommendation:** If RSMP 4 uses MQTT 5, mandate Message Expiry Interval for retained full updates to prevent stale data accumulation on the broker.

---

## 15. Summary of Recommendations

| # | Area | Recommendation | Priority |
|---|---|---|---|
| 1 | Attribute types | Formalize "Send on Change" and "Send Along" as first-class concepts. This solves the S0001/cyclecounter problem. | High |
| 2 | MQTT mapping | Define how streams map to MQTT topics. Full updates use retained messages; deltas do not. | High |
| 3 | QoS | Specify recommended QoS per stream type. | Medium |
| 4 | Delta encoding | Define delta format: attribute-level sparse updates. Keep it simple. | High |
| 5 | Rate limiting | Add minimum interval setting to prevent message flooding. | High |
| 6 | Stream discovery | Publish stream catalog as a retained message. | Medium |
| 7 | Stream control | Define start/stop as RSMP 4 commands with explicit responses. | Medium |
| 8 | Aggregation | Clarify window alignment (clock-aligned recommended) and partial windows. | Medium |
| 9 | Pruning | Define concrete pruning mechanism (presence-based or keep-alive). | Medium |
| 10 | Per-component vs bundled | Let SXLs decide per status type. Bundled for S0001 (atomicity), per-component for S0025. | Medium |
| 11 | Not-streamable statuses | Clearly identify statuses that should be request-response only (S0095, S0098, etc.). | Low |
| 12 | MQTT 5 | Consider Message Expiry and Topic Aliases if using MQTT 5. | Low |
| 13 | Administration limits | Define error responses for stream creation/modification failures. | Low |

---

## 16. Worked Example: S0001 in RSMP 4 Streams

To make this concrete, here is how S0001 would work end-to-end with the recommended stream model.

### Stream Definition (in SXL)

```yaml
stream:
  id: tlc.signalgroup.live
  code: tlc.signalgroup
  topic_suffix: null  # uses default status topic
  attributes:
    signalgroupstatus:
      type: on_change
    stage:
      type: on_change
    cyclecounter:
      type: send_along
    basecyclecounter:
      type: send_along
  update_rate: 5s       # full retained snapshot every 5 seconds
  delta_rate: on_change # delta on any on_change attribute transition
  min_interval: 100ms   # coalesce changes within 100ms
  default_state: off
  qos: 0
  prune_timeout: 300s
```

### Message Flow

**T=0.0s — Manager starts the stream and subscribes:**
```
Manager → Broker: SUBSCRIBE dk/cph/45fe/stream/live/tlc.signalgroup
Manager → Broker: PUBLISH  dk/cph/45fe/command/stream.start → {"stream": "tlc.signalgroup.live"}
```

**T=0.1s — Device publishes initial full update (retained):**
```
Topic: dk/cph/45fe/stream/live/tlc.signalgroup
Retain: true, QoS: 0
Payload: {
  "signalgroupstatus": "BBBA1NGBBBB",
  "stage": 3,
  "cyclecounter": 45.2,
  "basecyclecounter": 45.2
}
```

**T=3.7s — Signal group 5 transitions from green (1) to yellow (N):**
```
Topic: dk/cph/45fe/stream/live/tlc.signalgroup
Retain: false, QoS: 0
Payload: {
  "signalgroupstatus": "BBBANGBBBB",
  "cyclecounter": 48.9,
  "basecyclecounter": 48.9
}
```

Note: `stage` is omitted from the delta because it didn't change. `cyclecounter` and `basecyclecounter` are included because they're Send Along.

**T=5.0s — Periodic full update (retained):**
```
Topic: dk/cph/45fe/stream/live/tlc.signalgroup
Retain: true, QoS: 0
Payload: {
  "signalgroupstatus": "BBBANGBBBB",
  "stage": 3,
  "cyclecounter": 50.2,
  "basecyclecounter": 50.2
}
```

**T=5.0–35.0s — No signal changes for 30 seconds:**
Only periodic full updates at T=10, 15, 20, 25, 30, 35 (6 messages). In RSMP 3, this would have been 30 messages (at 1/s cyclecounter changes) or 300 messages (at 100ms changes).

### Result

- **Bandwidth savings:** 80–95% reduction compared to RSMP 3 with sendOnChange for S0001
- **Precision preserved:** cyclecounter is precisely tied to signal transitions
- **New subscriber experience:** Gets the latest full state immediately from retained message
- **Robustness:** Periodic full updates ensure eventual consistency even if deltas are lost (QoS 0)

---

## 17. Open Questions for Working Group

1. **Should RSMP 4 require MQTT 5 or remain compatible with MQTT 3.1.1?** Several features (message expiry, topic aliases) are valuable but MQTT 5 broker support is still not universal.

2. **Should stream IDs be globally unique or scoped per node?** Global uniqueness allows cross-referencing in a fleet management system but adds complexity.

3. **Should there be a "request last known value" mechanism for statuses that don't have a stream?** This would replace the RSMP 3 StatusRequest for on-demand data.

4. **How should RSMP 4 handle the transition from character-encoded strings (S0001 signalgroupstatus) to structured data?** Using an array of objects per signal group would be cleaner but breaks backward compatibility with tools that parse the character string.

5. **Should the stream concept explicitly support time-series databases?** Many deployments forward RSMP data to InfluxDB/TimescaleDB. If the stream payload includes a device-side timestamp, sequence number, and explicit measurement metadata, it becomes easier to ingest.
