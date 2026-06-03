---
key: PRODUCT-17449
title: Advanced Health Alerts, Warning signals and INFO events
---

# Advanced Health Alerts, Warning signals and INFO events

_Jira: [PRODUCT-17449](https://dt-rnd.atlassian.net/browse/PRODUCT-17449)_

## Short Abstract / Blogline

This VI provides additional  OOTB events which, per se,  bring immediate value as well as they provide an almost complete set of network related events.
The wew events are of two types: metric based events and the much needed, Syslog and trap based events.

Network device syslog and traps are semantic loaded but Dynatrace don't yet act on them.  E.g. BGP neighbor flaps, line-card resets, interface state changes, OSPF adjacency loss, hardware faults all surface in syslog when SNMP polling does not report it at all.

Additionnaly, but very strongly, the new  OOTB events bring value to the Network RCA VI (PRODUCT-15022) leading to better root cause analysis.

## Customer Zero

**Internal:** Dynatrace EDE. They already forwards syslog to Dynatrace. This batch of new events will help them adopt Network RCA.

**External:** Friendly customers  (Costo, DXC, SAFRAN ) currently running Dynatrace syslog but getting little value out of it. They have asked specifically for syslog alerting in Dynatrace so they can get more value out of syslog/trap ingestion.


## Target Audience

**Primary — SREs**:  Once an application problem involving service degradation is raised, they look at the attached network events. They make a determination on network innocence.
In the case where the network is the culprit, the network events attached help them to assess the root cause by themselves.

**Secondary — Network engineers:** Own the alert tuning and false-positive suppression for their environment. Need: visibility into which anomaly fired, ability to silence noisy  alerts per-device, and a way to validate that critical alert types reach the on-call team.

## Why and Competitive Context

**Why now:**

- Customers consolidating onto Dynatrace from SolarWinds/PRTG repeatedly cite syslog alerting as the last feature blocking decommission of the legacy tool.
- Support network RCA by providing an adequate set of HA&WS&INFO events

**Expected value:**

- For customers: eliminate a secondary syslog-alerting tool (Splunk-derived setups, SolarWinds Kiwi, Graylog), reduce MTTR on network-originated incidents by surfacing failures and info events that are not capture by SNMP or API polling.
- For Dynatrace: removes the most common objection in specialized network monitoring tools displacement deals: no or few OOTB alerts; increases stickiness of the Dynatrace with OOTB comprehensive health alerts.

**Key competitors:**

| Competitor | Approach | Our differentiation |
|---|---|---|
| **SolarWinds NPM + Kiwi Syslog** | Mature syslog parsing with a rule editor and per-vendor templates. Standalone tool, no link to application impact. | multi-purpose Anomaly detector flexibility |
| **Splunk** | Powerful  no out-of-the-box network rule packs. | Superior Dynatrace intellince solution. |
| **Datadog Network Device Monitoring** | Has syslog ingestion but limited OOTB rules | Superior Dynatrace intellince solution. |

## Use Case and User Journey

### Journey 1 — Onboarding a new device class

> "We just added a fleet of Arista switches. Make sure I get the same coverage I have on the Cisco ones."

1. **Configure syslog destination on the device** — runbook-driven, out of app scope.
2. **Open Syslog Sources in the Network App** — the new devices appear in the list once their first message is received. They are auto-bound to the matching device entity by source IP.
3. **Select and tune events** — the Arista EOS rule pack is not enabled by default. HA&WS are enabled and tuned from the Network/I&O App.

### Journey 2 — Application SRE

> "An application performance is affected by the network infrastructure, what should I fix to restore the application performance?"

1. **Problem notification** — the on-call SRE receives an application Davis problem. Network events are attached to the Problem. The application SRE does not react on network events not impacting the application. The relative severity of events attached to application problem helps me fix the most important problem first.

Network devices are not producing Health Alerts to generate. When an application problem is raised, the root cause analysis process find the root cause out all network events (INFO or Warning Signals).

2. **Device context** — the problem entity links directly to the network device in the  Network/I&O App. The Health Alerts column is populated with the "Fan failure" Health Alerts. The Events tab on Device Detail shows the originating syslog message verbatim, plus any related messages from the same device in the surrounding minutes.

### Journey 3 — Network operations

> "What should be done to restore the network desired state"

1. **From the device timeline** — the engineer opens Device Detail → Events for the failed device and sets the time window to the incident period.
2. **Full syslog stream** — every message received from the device in that window is shown, severity-coloured, with the ones that triggered a  Problem flagged.

### Journey 4 — Post-incident forensics

> "What did the device say in the 10 minutes before it failed?"

1. **From the device timeline** — the engineer opens Device Detail → Events for the failed device and sets the time window to the incident period.
2. **Full syslog stream** — every message received from the device in that window is shown, severity-coloured, with the ones that triggered a  Problem flagged.

### Application root cause analysis down to the network device

### Network operations

Based on the severity of the network device health degradation a Warning Signal or a Health Alert is raised.

## Functional Requirements

The event are classified based on the definitions:

### Health alerts

These are critical problems we identify out-of-the-box for customers. Detected health alerts mean the business is already impacted or at immediate risk of impact.
 As a developer, SRE, or ITOps practitioner, you should expect to be awakened in the middle of the night for such an alert.

### Warning signals

These are anomalous conditions that do not impact the business immediately.
As a developer, SRE, or ITOps practitioner, you want to be aware of these issues, but only during business hours.
Ideally, these events automatically trigger remediation actions. Warning signals are enabled out-of-the-box and have a “warning” severity.

Given the diversity of network or individual device criticality, the choice between health alerts and warning signal should be left to the end user.

### Event Catalog

| Feature | Health Alert | Warning Signal | Info | Source | Note |
| --- | :---: | :---: | :---: | --- | --- |
| Out-of-Maintenance Shutdown | ✓ | | | syslog | Device ceased offering connectivity service outside a maintenance window — immediate business impact |
| Power Supply Down | ✓ | | | syslog | Redundant power lost; device at immediate risk of full outage or lower capacity |
| Routing Engine Down | ✓ | | | syslog | Redundant control plane lost; device at immediate risk of loosing its control plane |
| Software Crash | ✓ | | | syslog | Device connectivity interrupted or at immediate risk |
| Fan Down | | ✓ | | syslog | Redundancy degraded but device still operational; risk of thermal shutdown if unaddressed |
| Out-of-Maintenance Reboot | | ✓ | | syslog | Device was temporarily offline; warrants investigation but service has recovered |
| CPU, Memory pressure Anomaly | | ✓ | | metric | Resource saturation degrades forwarding and convergence but does not immediately stop service |
| BGP Neighbor Down | | ✓ | | syslog | Routing session lost; redundant paths typically absorb the impact, but requires investigation |
| BGP Routing Table Anomaly | | ✓ | | metric | Unusual route churn or table size; no immediate outage but forwarding stability at risk |
| OSPF Neighbor Down | | ✓ | | syslog | Adjacency lost; traffic may reroute but convergence gap requires attention |
| EIGRP Neighbor Down | | ✓ | | syslog | Same as OSPF: adjacency loss with expected failover, not an immediate outage |
| IS-IS Neighbor Down | | ✓ | | syslog | Same as OSPF: adjacency loss with expected failover, not an immediate outage |
| Interface Contractual Saturation | | ✓ | | metric | Bandwidth threshold exceeded; degraded experience but no connectivity loss |
| Optical Module Alert or Warning | | ✓ | | syslog/metric | Signal degradation or threshold breach; link may be approaching failure but still operational |
| HSRP, VRRP failover | | ✓ | | syslog/metric | Gateway redundancy activated; primary path failed but service continues via standby |
| NTP synch loss | | ✓ | | syslog/metric | Time drift degrades log correlation and cert validation but does not impact packet forwarding |
| Temperature Anomaly | | ✓ | | metric/syslog | Device running hot; no immediate failure, but thermal shutdown risk if not addressed |
| Configuration Change | | | ✓ | Syslog, SNMP trap | Intentional or unintentional config edit; no anomaly by itself, recorded for audit and forensics |
| FRU Change | | | ✓ | syslog | Hardware insertion or removal; normal operational event, relevant for inventory tracking |
| Planned Reboot | | | ✓ | syslog | Scheduled restart within a maintenance window; expected, no anomaly |
| Planned Shutdown | | | ✓ | syslog | Scheduled shutdown within a maintenance window; expected, no anomaly |
| Software Change | | | ✓ | Anomaly detection on property | OS or firmware version change; informational record for change tracking and RCA context |

### Per vendor events

For all network device types supported. Execution priority based on the current revenue of supported or custom extensions.

- F5 (through DXS-3684)
- Cisco
- Netscaler
- Palo Alto
- Juniper
- Arista
- etc.

### Dynatrace intelligence integration

- Events appear in Application Problem pointing to a Service anomaly (slowness, error, low traffic)

## Non-Functional Requirements

**Performance:**

- Ingest sustained rate: expected volume nowhere near Dynatrace ingestion capacity.

**Scale:**

- Toal number of anomaly detectors estimation:  realistic guess: 20 device types, 30 event types per device type: 600 anomalies.

**Reliability:**

- Anomaly detection lifecycle support like other Gen3 events.

**Usability:**

- Default-off: to avoid cost explosion or alert fatigue. Add hints to enable the events to obtain  network RCA benefits.

---

## Out of Scope

- **Generic non-network syslog** (server OS, application syslog, security appliances). Different VI, different audience.
- **Cross-device events** (e.g., "BGP down on both sides of a link = link failure").

## Success Metrics

- **Adoption:** 80% of Dynatrace Network/I&O Apps customers with at least one health alerts or warning signals configured.
- **Problem volume:** number of Davis problems sourced from the events defined in this VI per month, segmented by Health Alerts.
- **Displacement:** number of customers who decommission a secondary syslog tool (Kiwi, Splunk pipeline, Graylog) within 12 months of enabling this feature.

## E2E Demo (for acceptance)

Demo are perform with help of syslog and SNMP simulator. E.g. MIMIC.
All generated events are test allow a protocol similar to the one below.

1. **Trigger:** simulate a "BGP neighbor down" event on a Cisco IOS edge router (syslog message: `%BGP-5-ADJCHANGE: neighbor 10.1.2.3 Down BGP Notification received`).
3. **Click through** to Device Detail. The Events tab shows the BGP message, plus the surrounding context messages from the device.
4. **Application Impact** link shows the downstream services that depend on this device's transit; confirm whether user sessions are affected.
5. **Resolution:** simulate the neighbor coming back up (`%BGP-5-ADJCHANGE: neighbor 10.1.2.3 Up`). The Davis problem closes automatically when the matching recovery rule fires within the dedupe window.
6. **Noise tuning sub-demo:** from a separate maintenance-induced fire, click "Tune this rule" → suppress for that device → confirm the next identical message is recorded but does not page.

---

## Enablement Requirements

- **Support:** runbooks anomaly covering "anomaly fired but I don't know what it means," "anomaly did not fire when expected,".  Decision tree for the most common device type messages.
- **Sales enablement:** displacement deck slide for SolarWinds Kiwi / Graylog / Splunk-syslog setups. Headline: same syslog, native to the device, paging the same on-call team as your app alerts.
- **Marketing:** brief launch blog focused on the SRE on-call experience (Journey 1). Cross-promote with the Network App launch beats.
