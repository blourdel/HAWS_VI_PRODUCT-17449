# Syslog based Health Alerts, Warning Signals and INFO events
This repo is defining Health Alerts, Warning Signals and INFO events for network devices.
The defined Health Alerts, Warning Signals and INFO events served in the network problem root cause identification process  . This folder (`/docs`) serves as a knowledge base for this application.

See [VI](./vi.md) for the product rationale, competitive context, and success metrics behind this app. Jira: [PRODUCT-17449](https://dt-rnd.atlassian.net/browse/PRODUCT-17449).

## Target users
1. **Network engineers** - are responsible for the health, reliability, performance, and cost of network devices, WAN(s), and other network services. For them the network is their daily domain.
2. **SREs** - are responsible for the overall health, reliability, performance, and cost of the IT environment. For them the network is a Tier-0 service -- if it is impacted, everything is impacted.

## Main use cases
1. **Network overview** - a quick overview of the health of all network devices and services.
3. **Devices & services** - a detailed inventory of all network devices, cloud network services, and WAN circuits.
4. **Application impact** - connects Real User Monitoring (RUM) with network data to understand which applications, regions, and individual users are impacted by network events.
5. **Onboarding** - enables the user to add monitoring in a highly automated way.