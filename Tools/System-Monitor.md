# System Monitor

> **Category:** System Performance Monitoring Tool
>
> **Platform:** Linux
>
> **License:** Open Source
>
> **Official Website:** https://help.gnome.org/

---

## Overview

System Monitor is the default graphical performance monitoring utility available in many Linux distributions. It provides real-time information about CPU, memory, network, storage, and process activity, allowing administrators to monitor overall system health and investigate abnormal resource utilization.

---

## Key Features

- CPU utilization monitoring
- Memory usage analysis
- Process management
- Network activity monitoring
- File system usage
- Resource visualization
- Process termination
- Real-time performance statistics

---

## Common Usage

Launch System Monitor:

```bash
gnome-system-monitor
```

Common workflow:

1. Launch System Monitor.
2. Monitor resource utilization.
3. Observe active processes.
4. Identify abnormal activity.
5. Analyze system performance.

---

## CEH Practical Example

During **Module 10 – Denial-of-Service**, System Monitor was used on the Ubuntu target machine to observe the increase in CPU and memory utilization caused by the simulated botnet-based DDoS attack. The tool provided real-time performance metrics demonstrating the operational impact of the attack.

---

## Defensive Perspective

System performance monitoring helps security teams detect abnormal resource consumption that may indicate denial-of-service attacks, malware activity, or other security incidents affecting system stability.

---

## Used In

- Module 10 – Denial-of-Service

---

## References

- Official Documentation: https://help.gnome.org/users/gnome-system-monitor/