# Alerting doctrine


## Alerting definitions

### Health alerts

These are critical problems we identify out-of-the-box for customers. Detected health alerts mean the business is already impacted or at immediate risk of impact.
 As a developer, SRE, or ITOps practitioner, you should expect to be awakened in the middle of the night for such an alert.

### Warning signals

These are anomalous conditions that do not impact the business immediately.
As a developer, SRE, or ITOps practitioner, you want to be aware of these issues, but only during business hours.
Ideally, these events automatically trigger remediation actions. Warning signals are enabled out-of-the-box and have a “warning” severity.

### Custom alerts

These are customer defined alerts, that represent their expert insights or opinions, whereas Health alerts and Warning signals are Dynatrace’s expert insights and opinions. Similarly, the cost of custom alerts resides with the customer, whereas the cost of Health alerts and Warning signals resides with the Observability solution.

### Alert templates

These are suggestions for how a customer could extend their observability further. They should not be required to identify and remediate business impact. They are instantiated as custom alerts when the customer uses them.

## Network definitions

### Network area

A group of network devices, physically connected to each other, that provides a connectivity service.

### Redundancy

It is desirable to provide redundancy for all devices of a network area.

### Critical device

A critical device is the only one to provide a given function in a network area.
If a critical device belonging to a network area is not providing service, it impacts all or a part of the network area clients.

### Principles

- Health Alerts and Warning signals apply to individual network devices not to a network area.
- While a network area may provide redundancy, some of its members are not redundant. E.g. device directly connecting client of the network area service.
- A lack of redundancy in a network area impacts partially the service it provides.
- Health Alert is raised only in these situations:
  - a critical device ceased offering connectivity service
  - a critical device redundancy status is degraded (redundant power lost, redundant fan tray down)

## Use cases

### Application root cause analysis down to the network device

Network devices are not producing Health Alerts. When an application problem is raised, the root cause analysis process find the root cause out all network events (INFO or Warning Signals).

### Network operations

Based on the severity of the network device health degradation a Warning Signal or a Health Alert is raised.
