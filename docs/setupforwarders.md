# Setting Up Splunk Universal Forwarders for Gaming Servers

This guide walks you through installing and configuring Splunk Universal Forwarders (UF) on your game servers to collect and forward logs, metrics, and in-game events to your Splunk deployment.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Installing the Universal Forwarder](#installing-the-universal-forwarder)
- [Initial Universal Forwarder Configuration](#initial-universal-forwarder-configuration)
- [Configuring Data Inputs](#configuring-data-inputs)
- [Securing the Forwarder](#securing-the-forwarder)
- [Connecting to Indexers](#connecting-to-indexers)
- [Verifying Data Flow](#verifying-data-flow)
- [Troubleshooting](#troubleshooting)
- [References](#references)

---

## Overview

The Splunk Universal Forwarder is a lightweight agent that collects data from your game servers and securely forwards it to Splunk indexers. This ensures only the forwarder communicates with your Splunk deployment, keeping your game servers decoupled from the analytics layer.

---

## Prerequisites

- Access to the game server(s) with sufficient privileges to install software
- Splunk indexer address and port (default: 9997)
- Splunk admin credentials for initial setup
- Network connectivity between the game server and the Splunk indexer

---

## Installing the Universal Forwarder

### Download the Installer

- [Download Splunk Universal Forwarder](https://www.splunk.com/en_us/download/universal-forwarder.html)

Choose the appropriate package for your operating system (Linux, Windows, etc.).

### Install on Linux (example)

```bash
wget -O splunkforwarder.tgz 'https://download.splunk.com/products/universalforwarder/releases/<version>/linux/splunkforwarder-<version>-<build>-Linux-x86_64.tgz'
tar -xvzf splunkforwarder.tgz -C /opt
cd /opt/splunkforwarder
./bin/splunk start --accept-license --answer-yes --no-prompt
```

### Install on Windows (example)
- Download the `.msi` installer and run it with administrator privileges.

---

## Initial Universal Forwarder Configuration

### Set Administrator Credentials

```bash
./bin/splunk enable boot-start
./bin/splunk set deploy-poll <deployment-server>:8089
```

Or, for basic standalone setup:

```bash
./bin/splunk add forward-server <indexer-host>:9997
```

---

## Configuring Data Inputs

**Main files:**  
- `$SPLUNK_HOME/etc/system/local/inputs.conf`

### Example: Monitor Game Server Logs

```ini
[monitor:///var/log/game-server/*.log]
disabled = false
sourcetype = game-server-log

[monitor:///var/log/game-events/*.log]
disabled = false
sourcetype = game-event-log
```

- Copy your edited `inputs.conf` to the appropriate directory and restart the forwarder:

```bash
./bin/splunk restart
```

---

## Securing the Forwarder

- **Enable SSL forwarding** for secure data transmission.
- Edit `outputs.conf` and set:

```ini
[tcpout]
defaultGroup = default-indexers

[tcpout:default-indexers]
server = <indexer-host>:9997
sslCertPath = $SPLUNK_HOME/etc/auth/mycert.pem
sslRootCAPath = $SPLUNK_HOME/etc/auth/cacert.pem
sslPassword = <password>
sslVerifyServerCert = true
```

- Generate or obtain SSL certificates as needed.

---

## Connecting to Indexers

- Add your indexers (single or clustered):

```bash
./bin/splunk add forward-server <indexer1>:9997
./bin/splunk add forward-server <indexer2>:9997
```

- Confirm using:

```bash
./bin/splunk list forward-server
```

---

## Verifying Data Flow

- On the Splunk Enterprise/Search Head, search for incoming data:

```spl
index=* host=<your-game-server-hostname>
```

- Check the forwarder status:

```bash
./bin/splunk list monitor
./bin/splunk show splunkd-log
```

---

## Troubleshooting

- **No data received?** Check firewalls, network connectivity, and forwarder logs (`$SPLUNK_HOME/var/log/splunk/splunkd.log`).
- **Indexers not listed?** Re-add with correct addresses and ensure ports are open.
- **Permission issues?** Run as an appropriate user with access to log files.

---

## References

- [Splunk Universal Forwarder Documentation](https://docs.splunk.com/Documentation/Forwarder)
- [inputs.conf Spec](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Inputsconf)
- [outputs.conf Spec](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Outputsconf)
