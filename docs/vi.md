---
key: PRODUCT-17449
title: Advanced Health Alerts, Warning signals and INFO events
---

# Advanced Health Alerts, Warning signals and INFO events

_Jira: [PRODUCT-17449](PRODUCT-17449https://dynatrace.atlassian.net/browse/)_

## Short Abstract / Blogline

This VI provides additional events to bring value to the Network RCA VI (PRODUCT-15022) leading to better root cause analysis. It adds additional metric based events together with the much needed, Syslog and trap based events.
Network device syslog and traps are semantic loaded but we don't yet act on them. BGP neighbor flaps, line-card resets, interface state changes, OSPF adjacency loss, hardware faults all surface in syslog when SNMP polling does not report it at all. Today Dynatrace ingests these messages as logs but does not translate them into events.

This VI delivers OOTB syslog parsing, vendor rule packs, and a Davis event pipeline that turns critical syslog and trap messages into actionable on-call problems correlated with application impact.
It adds a few metric based events that were excluded first iteration of the HA&WS for network device project.



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

### Journey 2 — SRE incident triage 

> "An alert fired about a network device. What broke, where, and is it affecting users?"

1. **Problem notification** — the on-call SRE receives a Davis problem titled e.g. "FAN fails for  edge-rtr-fra-01". The problem opens to the standard Davis problem view.
2. **Device context** — the problem entity links directly to the network device in the  Network/I&O App. The Health Alerts column is populated with the "Fan failure" Health Alerts. The Events tab on Device Detail shows the originating syslog message verbatim, plus any related messages from the same device in the surrounding minutes.

### Journey 3 — Post-incident forensics

> "What did the device say in the 10 minutes before it failed?"

1. **From the device timeline** — the engineer opens Device Detail → Events for the failed device and sets the time window to the incident period.
2. **Full syslog stream** — every message received from the device in that window is shown, severity-coloured, with the ones that triggered a  Problem flagged.

## Functional Requirements

### Event Catalog

| Feature | Health Alert | Warning Signal | Source |
| --- | :---: | :---: | --- |
| FRU Down | ✓ | | syslog |
| Fan Down | ✓ | | syslog |
| Power Supply Down | ✓ | | syslog |
| Routing Engine Down | ✓ | | syslog |
| Software Crash | ✓ | | syslog |
| Temperature Anomaly | ✓ | | metric/syslog |
| BGP Neighbor Down | | ✓ | syslog |
| BGP Routing Table Anomaly | | ✓ | metric |
| Control Plane Pressure Anomaly | | ✓ | metric |
| EIGRP Neighbor Down | | ✓ | syslog |
| IS-IS Neighbor Down | | ✓ | syslog |
| Interface Contractual Saturation | | ✓ | metric |
| OSPF Neighbor Down | | ✓ | syslog |
| Optical Module Alert | | ✓ | syslog/metric |
| Out-of-Maintenance Reboot | | ✓ | syslog |
| Out-of-Maintenance Shutdown | | ✓ | syslog |
| Configuration Change | | | Syslog, SNMP trap |
| FRU Change | | | syslog |
| Planned Reboot | | | syslog |
| Planned Shutdown | | | syslog |
| Software Change | | | Anomaly detection on property |

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
