# The Case for RSMP 4

## Summary
RSMP 4 is the next-generation core specification for RSMP (Road Side Message Protocol). Built on a modern messaging backbone build on on top of MQTT, RSMP 4 modernises how traffic management systems, field equipment and vendors communicate. Work on the draft is mandated by the RSMP Steering Group and is available as an online draft: https://rsmp-nordic.github.io/rsmp_core_v4/.

This document summarises what RSMP 4 is, why it is needed, and the concrete benefits it delivers to road authorities across Europe.

## What is RSMP 4?
- A core protocol specification defining message formats, topics/concepts and interoperability rules for traffic control equipment and systems.
- Designed around contemporary, lightweight publish/subscribe messaging (MQTT) to support both on-premises and cloud-native deployments.
- An evolution of the RSMP family (building on lessons from RSMP 3) with an emphasis on scalability, security, and easier integration between vendors and authorities.
- The specification is developed and coordinated by RSMP Nordic; details and draft work are tracked in the project repository and related documentation (see links at the end).

## Why RSMP 4 is needed
- Legacy integration complexity: Many traffic control projects today rely on bespoke or older protocols that are costly and slow to integrate across multiple vendors and jurisdictions.
- Cloud and IoT shift: Traffic management solutions increasingly use cloud services, edge devices and IoT sensors. A messaging-first core (MQTT-based) better supports these architectures.
- Scalability demands: Urban mobility, smart city initiatives and multi-modal transport systems require a protocol that scales easily from single intersections to city- and region-wide deployments.
- Security & resilience: Modern cryptographic standards, authenticated connections, and better session/connection semantics are required for trustworthy operations.
- Faster innovation: Standardised, vendor-neutral interfaces reduce duplication of work and lower the barrier for new services, analytics and third‑party integrations.

## Benefits for European road authorities
- Vendor neutrality and interoperability
  - Easier procurement and integration when equipment from different vendors speaks the same core language.
  - Reduced vendor lock-in and clearer interoperability testing points.
- Lower total cost of ownership
  - Standardised messaging reduces bespoke integration effort, shortening project timelines and reducing maintenance costs.
  - Reusable tooling, libraries and cloud-native deployment patterns mean faster rollouts and cheaper upgrades.
- Scalability & future-proofing
  - Support for MQTT and modern messaging patterns enables scaling from single junctions to regional infrastructures and hybrid edge/cloud topologies.
  - Facilitates addition of new sensors, ITS services and mobility applications without redesigning core communications.
- Improved security and operations
  - Modern transport and authentication (TLS, certificate management, token-based auth, etc.) raise the baseline security for connected field devices.
  - Better connection and session semantics help with resilient operations over lossy or constrained networks.
- Better data and service integration
  - Consistent message models make it simpler to aggregate telemetry and events for analytics, predictive maintenance, and traffic optimisation.
  - Easier integration with third‑party systems (traffic information services, C-ITS platforms, emergency services).
- Cross-border and regional collaboration
  - Shared standards simplify cooperation across municipal, regional and national boundaries — valuable for pan-European transport corridors and cross-border traffic management.
