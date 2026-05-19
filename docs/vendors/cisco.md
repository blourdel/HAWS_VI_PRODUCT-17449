## Cisco IOS System Message Guides:

cisco web site:  https://content.cisco.com

### IOS XE 17.x (Catalyst 9600): https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9600/software/release/17-17/configuration_guide/sys_mgmt/
### IOS 15.x SY: https://www.cisco.com/c/en/us/td/docs/ios/15_0sy/system/messages/15sysmg.pdf
### IOS XE ISR 4000: https://www.cisco.com/c/en/us/td/docs/routers/access/4400/software/configuration/xe-16-9/isr4400swcfg-xe-16-9-book/system_messages.html

### Specifics
Here's a comprehensive reference guide for Cisco syslog messages related to system events like reboot, configuration changes, and crashes.

---

## Cisco Syslog Messages: System Events Reference

### Message Format

Cisco IOS syslog messages follow this format:
`seq no:timestamp: %FACILITY-SEVERITY-MNEMONIC: description`

Severity levels range from 0 (Emergency) to 7 (Debug) — the lower the number, the more critical the event. Alerts and emergencies indicate serious problems like a router running out of memory or a process crashing; critical, error, and warning messages cover important events like interfaces going down.

---

### Key System Event Messages (`%SYS` Facility)

These are the most relevant messages for the events you're tracking:

| Syslog Message | Severity | Description |
|---|---|---|
| `%SYS-5-RESTART` | 5 (Notice) | System restarted/rebooted |
| `%SYS-6-BOOTTIME` | 6 (Info) | System boot time recorded |
| `%SYS-5-CONFIG_I` | 5 (Notice) | Configuration change made |
| `%SYS-5-RELOAD` | 5 (Notice) | Reload requested |
| `%SYS-3-CPUHOG` | 3 (Error) | CPU usage spike (process hogging CPU) |
| `%SYS-2-MALLOCFAIL` | 2 (Critical) | Memory allocation failure |
| `%SYS-0-SYS_LCPERR` | 0 (Emergency) | Fatal system error |

You can view a count of these events with the `show logging count` command, which breaks down occurrences by facility and mnemonic — for example, `SYS BOOTTIME`, `SYS RESTART`, and `SYS CONFIG_I` all appear as separate rows with timestamps of last occurrence.

---

### Configuration Change Message

`%SYS-5-CONFIG_I` is the standard message seen when a device is configured — for example: `%SYS-5-CONFIG_I: Configured from console by console`

---

### Process Crash / System Report Messages

When a device generates a System Report (which can occur due to a crash, a process restarting too many times, or a memory issue), the message `%PMAN` or a system report notification appears. This can also be triggered by repeated process failures or power issues.

Key crash-related messages:

| Syslog Message | Severity | Description |
|---|---|---|
| `%PMAN-0-PROCESS_NOTIFICATION` | 0 (Emergency) | Process lifecycle notification failure |
| `%PMAN-6-PROCSHUT` | 6 (Info) | A process gracefully shut down |
| `%PMAN-6-PROCSTART` | 6 (Info) | A process started |
| `%SYS-2-MALLOCFAIL` | 2 (Critical) | Memory allocation failure, often precedes crash |

---

### Official Cisco Reference Tools & Documentation

**1. Cisco Error Message Decoder (EMD)** — the primary lookup tool:
Copy any error message from your console or syslog and paste it into the tool at **https://www.cisco.com/cgi-bin/Support/Errordecoder/index.cgi**. It returns an explanation, 
