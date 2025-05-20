# Server-Side Observability Setup for Gaming Servers

This guide explains how to configure Splunk Universal Forwarders and your gaming server environment to collect critical server-side metrics and logs. These insights help monitor server health, performance, and stability.

---

## Table of Contents

- [Overview](#overview)
- [What to Monitor](#what-to-monitor)
- [Configuring System Metrics Collection](#configuring-system-metrics-collection)
- [Monitoring Game Server Logs](#monitoring-game-server-logs)
- [Example Splunk Configurations](#example-splunk-configurations)
- [Best Practices](#best-practices)
- [References](#references)

---

## Overview

Server-side observability focuses on collecting and analyzing metrics and logs related to the underlying hardware and OS running your game server(s). This includes CPU, memory, disk, and network statistics, as well as OS and application logs.

---

## What to Monitor

- **CPU Usage:** Track utilization, spikes, and sustained high load.
- **Memory Usage:** Monitor for leaks, high usage, and swap activity.
- **Disk Usage & I/O:** Watch for low disk space and high I/O wait.
- **Network Throughput & Latency:** Ensure healthy data flow and low packet loss.
- **Operating System Logs:** Capture system errors, warnings, or hardware failures.
- **Game Server Logs:** Collect crash reports, error logs, and performance data from the game server application.

---

## Configuring System Metrics Collection

### Using Splunk Add-on for Unix or Windows

- **Linux/Unix:**  
  Install the [Splunk Add-on for Unix and Linux](https://splunkbase.splunk.com/app/833/) on the forwarder.

  Add to `inputs.conf`:
  ```ini
  [script://./bin/ps.sh]
  interval = 60
  sourcetype = ps
  disabled = 0

  [script://./bin/vmstat.sh]
  interval = 60
  sourcetype = vmstat
  disabled = 0
  ```

- **Windows:**  
  Install the [Splunk Add-on for Microsoft Windows](https://splunkbase.splunk.com/app/742/).

  Example for `inputs.conf`:
  ```ini
  [WinHostMon://CPU]
  type = CPU
  collectionInterval = 60

  [WinHostMon://Memory]
  type = Memory
  collectionInterval = 60
  ```

### Using Built-In Forwarder Monitors

- To monitor system logs:
  ```ini
  [monitor:///var/log/messages]
  disabled = false
  sourcetype = syslog

  [monitor:///var/log/syslog]
  disabled = false
  sourcetype = syslog
  ```

---

## Monitoring Game Server Logs

- Identify your game server’s log file locations (e.g., `/var/log/game-server/`, `/opt/game/server/logs/`).
- Add file monitors to the forwarder's `inputs.conf`:
  ```ini
  [monitor:///var/log/game-server/*.log]
  disabled = false
  sourcetype = game-server-log

  [monitor:///opt/game/server/logs/*.log]
  disabled = false
  sourcetype = game-server-log
  ```

- For crash dumps or error logs:
  ```ini
  [monitor:///var/log/game-server/crash/*.log]
  disabled = false
  sourcetype = game-server-crash
  ```

---

## Example Splunk Configurations

- **Sample `inputs.conf`:**
  ```ini
  # System metrics
  [script://./bin/ps.sh]
  interval = 60
  sourcetype = ps
  disabled = 0

  # System logs
  [monitor:///var/log/syslog]
  disabled = false
  sourcetype = syslog

  # Game server logs
  [monitor:///var/log/game-server/*.log]
  disabled = false
  sourcetype = game-server-log
  ```

- After editing, restart the forwarder:
  ```bash
  ./bin/splunk restart
  ```

---

## Best Practices

- Use unique `sourcetype` values for each log/metric source for easier searching and dashboarding.
- Monitor only necessary files to reduce noise and resource usage.
- Regularly review monitored paths as game server updates may add or move log locations.
- Use descriptive names for servers/hosts in Splunk for easy identification in dashboards.

---

## References

- [Splunk Add-on for Unix and Linux](https://splunkbase.splunk.com/app/833/)
- [Splunk Add-on for Microsoft Windows](https://splunkbase.splunk.com/app/742/)
- [inputs.conf documentation](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Inputsconf)
