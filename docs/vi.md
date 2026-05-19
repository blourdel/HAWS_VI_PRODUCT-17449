# Network Device Syslog Alerting — Value Increment

_Jira: [PRODUCT-17449](https://dynatrace.atlassian.net/browse/PRODUCT-17449)_

## Short Abstract / Blogline

**Internal:** Network device syslog is the highest-signal failure stream we don't yet act on. BGP neighbor flaps, line-card resets, interface state changes, OSPF adjacency loss, hardware faults, and AAA failures all surface in syslog first — usually seconds before SNMP polling catches the resulting state change. Today Dynatrace ingests these messages as logs but does not translate them into Davis problems bound to the originating device entity. This VI delivers OOTB syslog parsing, vendor rule packs, and a Davis event pipeline that turns critical syslog messages into actionable on-call problems correlated with application impact.

**External:** When a switch loses an uplink at 2am, the on-call responder shouldn't have to grep through raw log streams to find out. Network device syslog tells you what broke, on which interface, with which neighbor, in plain text — but only if your monitoring tool understands it. The Dynatrace Network App now turns syslog into Davis problems automatically: the same alert that says "BGP neighbor down" also tells you which applications routed through that device and which users are seeing errors right now.

**Strategic fit:** Closes a known coverage gap in the [[Dynatrace Network App]] VI. Companion to the SNMP, flow, and path-monitoring data sources — syslog is the fourth pillar customers expect from any serious network monitoring product. Required for credible displacement of SolarWinds NPM, PRTG, and LogicMonitor, all of which ship syslog alerting out of the box. Also strengthens MTTR claims in competitive deals against Datadog NPM.

---

## Customer Zero

**Internal:** Dynatrace EDE. EDE already forwards syslog from corporate network devices to a separate collector for forensic review but has no automated alerting on it. They are the ideal Customer Zero: real network devices, real on-call rotation, and a clear "before/after" story we can measure (number of incidents where syslog evidence was found post-hoc vs. surfaced as a Davis problem in real time).

**External:** Friendly customers _[confirm: name 2–3 design partners]_ currently running Dynatrace network monitoring alongside a legacy syslog tool (typically Splunk-based or SolarWinds Kiwi Syslog). They have asked specifically for syslog alerting in Dynatrace so they can decommission the secondary tool.

---

## Target Audience

**Primary — SREs / on-call responders:** Receive the Davis problems triggered by syslog, triage them, and decide whether the problem affects users. Need: clear problem titles, the originating device, the raw syslog message, and a direct link to Application Impact. Do not configure parsers or write rules.

**Secondary — Network engineers:** Own the rule pack tuning and false-positive suppression for their environment. Need: visibility into which rule fired, ability to silence noisy rules per-device or per-severity, and a way to validate that critical message types reach the on-call team.

---

## Why and Competitive Context

**Why now:**
- The [[Dynatrace Network App]] ships device health from SNMP polling, but polling intervals (typically 60s) mean device-state changes are detected 30–60 seconds later than the syslog message that announced them. For incident triage, that gap is the difference between proactive and reactive response.
- High-signal failure modes (BGP neighbor loss, line-card fault, fan failure, AAA brute-force) are visible **only** in syslog — SNMP polling will never see them.
- Customers consolidating onto Dynatrace from SolarWinds/PRTG repeatedly cite syslog alerting as the last feature blocking decommission of the legacy tool.

**Expected value:**
- For customers: eliminate a secondary syslog-alerting tool (Splunk-derived setups, SolarWinds Kiwi, Graylog), reduce MTTR on network-originated incidents by surfacing failures at message-receipt time instead of poll-cycle time.
- For Dynatrace: removes the most common objection in Network App displacement deals; increases stickiness of the Network App by adding a data source most competitors charge separately for. Target: _[confirm: e.g., 30% of Network App customers enable syslog within 6 months of GA]_.

**Key competitors:**

| Competitor | Approach | Our differentiation |
|---|---|---|
| **SolarWinds NPM + Kiwi Syslog** | Mature syslog parsing with a rule editor and per-vendor templates. Standalone tool, no link to application impact. | Davis problems bound to the same device entity as SNMP/flow data — one timeline, one alert, automatic application impact correlation. No second tool to maintain. |
| **Splunk / Graylog (DIY)** | Powerful search but no out-of-the-box network rule packs; requires customer-built dashboards and alerts. | OOTB rule packs for Cisco IOS/NX-OS, Junos, Arista EOS — works on day one without writing parsers. |
| **Datadog Network Device Monitoring** | Has syslog ingestion but limited OOTB rules; alerting is monitor-by-monitor manual config. | Davis AI severity classification — message → problem grade is automatic, not a per-rule config job. |

---

## Use Case and User Journey

### Journey 1 — SRE incident triage (primary)

> "An alert fired about a network device. What broke, where, and is it affecting users?"

1. **Davis problem notification** — the on-call SRE receives a Davis problem titled e.g. "BGP neighbor 10.1.2.3 down on edge-rtr-fra-01". The problem opens to the standard Davis problem view.
2. **Device context** — the problem entity links directly to the network device in the [[Dynatrace Network App]]. The Events tab on Device Detail shows the originating syslog message verbatim, plus any related messages from the same device in the surrounding minutes.
3. **Application Impact** — from the device, the SRE follows the Application Impact link to see which applications and user sessions route through the affected device. If user impact is confirmed, the SRE escalates; if not, the issue is handed to network engineering without paging the application owner.

### Journey 2 — Network engineer noise tuning

> "This message fires every time we do maintenance. Stop paging me for it."

1. **Open the problem's rule** — from any syslog-triggered Davis problem, a "Tune this rule" link opens the rule definition (which syslog pattern triggered it, what severity it mapped to).
2. **Suppress scope** — the engineer suppresses the rule for either (a) a specific device, (b) a device group/tag, or (c) a maintenance window. Suppressions are stored in app config, not hidden in user-specific filters.
3. **Validation** — the rule list shows fire count over the last 7 days per rule, so engineers can spot rules that are now silent because they were over-suppressed.

### Journey 3 — Onboarding a new device class

> "We just added a fleet of Arista switches. Make sure I get the same coverage I have on the Cisco ones."

1. **Configure syslog destination on the device** — runbook-driven, out of app scope, but documented in onboarding.
2. **Open Syslog Sources in the Network App** — the new devices appear in the list once their first message is received. They are auto-bound to the matching device entity by source IP.
3. **Confirm rule pack** — the Arista EOS rule pack is enabled by default; the engineer sees fire counts in the rule list and verifies that the same critical message types they care about (interface flap, MLAG peer down, etc.) have active rules.

### Journey 4 — Post-incident forensics

> "What did the device say in the 10 minutes before it failed?"

1. **From the device timeline** — the engineer opens Device Detail → Events for the failed device and sets the time window to the incident period.
2. **Full syslog stream** — every message received from the device in that window is shown, severity-coloured, with the ones that triggered a Davis problem flagged. No need to leave the app for raw log search.

---

## Functional Requirements

### Syslog ingestion
- RFC 3164 (BSD) and RFC 5424 (structured) parsing.
- UDP/514 and TCP/6514 (TLS) listeners. _[confirm: do we ship a dedicated collector, or rely on OneAgent / OTel collector for ingest?]_
- Source-IP-based binding to a `EXT_NETWORK_DEVICE` entity. Hostname fallback when reverse DNS is configured. Unbound messages are stored but flagged in a "Unbound sources" view for ops follow-up.

### Vendor rule packs (GA scope)
- **Cisco IOS / IOS-XE**
- **Cisco NX-OS**
- **Juniper Junos**
- **Arista EOS**
- _[confirm: include Palo Alto PAN-OS at GA, or defer to post-GA?]_

Each rule pack ships:
- A curated set of "always alert" message patterns (BGP/OSPF adjacency, interface down on uplinks, hardware faults, AAA failures).
- A "informational, do not alert" pattern set (e.g., regular SNMP poll auth from monitoring stations) so noisy benign messages are filtered before they hit the problem stream.
- Default severity → Davis problem grade mapping.

### Rule engine
- Pattern matching: literal substring, regex, and structured-field equality (for RFC 5424).
- Per-rule controls: enabled, severity, problem title template, dedupe window.
- Per-device and per-tag suppression overrides.
- Per-rule fire-count metrics surfaced in the rule list UI.

### UI surfaces
- **Network App → Device Detail → Events tab:** integrate syslog into the existing events stream. Already-planned tab; this VI adds syslog as a source.
- **Network App → Syslog Sources** (new view): list of bound and unbound syslog sources, last-message timestamp, message rate, rule pack assignment.
- **Network App → Syslog Rules** (new view): rule list with fire counts, enabled state, suppressions.

### Davis integration
- Each fire creates a Davis event of grade matching the rule's mapping.
- Repeat fires within the dedupe window attach to the existing problem rather than creating a new one.
- Problem entity is the network device, so existing Davis correlation (application services downstream) works without additional wiring.

---

## Non-Functional Requirements

**Performance:**
- Ingest sustained rate: _[confirm with engineering: e.g., 10,000 messages/sec per environment]_.
- Time from message receipt to Davis problem creation: < 30 seconds at P95.
- Device Detail → Events tab must render the last 1 hour of syslog for a device within 3 seconds.

**Scale:**
- Up to 10,000 monitored devices per environment (matches the [[Dynatrace Network App]] scale target).
- Hot retention 30 days for raw syslog; cold retention follows platform log defaults.

**Reliability:**
- No message loss for severity ≤ Error under normal load. Drop policy for Info/Debug above ingest threshold must be configurable and observable.
- Rule pack updates are versioned. Existing suppressions survive a rule pack version bump.
- Partial degradation: if the syslog pipeline is down, the rest of the Network App must continue functioning (device health from SNMP, flow data, etc.).

**Usability:**
- Default-on for new Network App customers: the GA rule packs are enabled the moment the first syslog message is received from a recognised device class. No "configure 200 rules to get value" experience.

---

## Out of Scope

- **Custom user-written parsers at GA.** Customers use the OOTB rule packs only. Custom rules considered for a follow-up VI once we see how the OOTB pack lands.
- **Syslog forwarder configuration on the devices themselves.** Documented in onboarding runbooks; we do not push config to devices.
- **Generic non-network syslog** (server OS, application syslog, security appliances). Different VI, different audience.
- **Notification routing config.** Uses the standard Dynatrace platform notification rules; this VI does not introduce app-specific notification destinations.
- **Historical replay** of pre-existing syslog archives. Ingest is live-stream only at GA.
- **Cross-device correlation rules** (e.g., "BGP down on both sides of a link = link failure"). Davis problem grouping handles correlation; we do not ship app-specific correlation logic.

---

## Success Metrics

- **Adoption:** % of [[Dynatrace Network App]] customers with at least one device sending syslog within 90 days of GA. Target: _[confirm: 30%]_.
- **Problem volume:** number of Davis problems sourced from syslog per month, segmented by rule pack. Demonstrates the rule packs are actually firing on real events.
- **MTTR delta:** for incidents involving a network device, compare time-to-acknowledge for syslog-sourced problems vs. SNMP-sourced problems. Expect syslog-sourced problems to be acknowledged measurably earlier (the device announced itself faster).
- **Displacement:** number of customers who decommission a secondary syslog tool (Kiwi, Splunk pipeline, Graylog) within 12 months of enabling this feature.
- **Customer Zero:** EDE decommissions their separate syslog collector and uses only Dynatrace Network App for syslog-based alerting.
- **Noise health:** ratio of fired problems to acknowledged problems per rule. Rules with very low acknowledgment rates are candidates for rule-pack revision.

---

## E2E Demo (for acceptance)

Demo runs on the internal EDE tenant, using a real device or a syslog generator playing recorded messages.

1. **Trigger:** simulate a BGP neighbor down event on a Cisco IOS edge router (syslog message: `%BGP-5-ADJCHANGE: neighbor 10.1.2.3 Down BGP Notification received`).
2. **Davis problem appears** within 30 seconds, titled "BGP neighbor 10.1.2.3 Down on edge-rtr-fra-01", grade Error.
3. **Click through** to Device Detail. The Events tab shows the BGP message, plus the surrounding context messages from the device.
4. **Application Impact** link shows the downstream services that depend on this device's transit; confirm whether user sessions are affected.
5. **Resolution:** simulate the neighbor coming back up (`%BGP-5-ADJCHANGE: neighbor 10.1.2.3 Up`). The Davis problem closes automatically when the matching recovery rule fires within the dedupe window.
6. **Noise tuning sub-demo:** from a separate maintenance-induced fire, click "Tune this rule" → suppress for that device → confirm the next identical message is recorded but does not page.

---

## Enablement Requirements

- **Support:** runbooks for the four GA rule packs covering "rule fired but I don't know what it means," "rule did not fire when expected," and "syslog source bound to the wrong device." Decision tree for the most common Cisco/Junos/Arista message types.
- **Licensing:** _[confirm: included in Network App entitlement or separately metered on ingest volume? Sales positioning depends on this.]_
- **Sales enablement:** displacement deck slide for SolarWinds Kiwi / Graylog / Splunk-syslog setups. Headline: same syslog, native to the device, paging the same on-call team as your app alerts.
- **Marketing:** brief launch blog focused on the SRE on-call experience (Journey 1). Cross-promote with the [[Dynatrace Network App]] launch beats.
- **Rollout:** GA as part of the next Network App release. No staged cohort. Customer Zero (EDE) must validate before signoff. Rule packs are independently versioned and can be patched between Network App releases without a full app redeploy.