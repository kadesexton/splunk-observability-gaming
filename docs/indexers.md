# Splunk Indexers for Gaming Server Observability

This guide covers best practices and configuration steps for setting up Splunk Indexers to store, manage, and make searchable all telemetry, metrics, and logs sent from your game servers via Universal Forwarders.

---

## Table of Contents

- [Overview](#overview)
- [Indexer Roles in Splunk](#indexer-roles-in-splunk)
- [Sizing and Scaling Indexers](#sizing-and-scaling-indexers)
- [Indexer Cluster Setup (Distributed Architecture)](#indexer-cluster-setup-distributed-architecture)
- [Best Practices](#best-practices)
- [References](#references)

---

## Overview

Splunk Indexers are responsible for:

- Ingesting data sent from Universal Forwarders and other inputs
- Parsing, indexing, and storing events, logs, and metrics
- Enabling fast search and retrieval for dashboards, alerts, and analytics

A well-configured indexer layer is critical for performance and reliability, especially for high-volume gaming environments.

---

## Indexer Roles in Splunk

- **Standalone Indexer:**  
  Used for smaller deployments or local testing. Handles indexing and searching.
- **Indexer Cluster:**  
  Used in production/distributed setups for redundancy, scalability, and high availability.
  - **Cluster Master (Manager):** Orchestrates the cluster, manages configuration and replication.
  - **Peer Nodes (Indexers):** Store and replicate data.
  - **Search Head(s):** Connects to the cluster for distributed search and dashboards.

---

## Sizing and Scaling Indexers

- **Estimate Daily Data Volume:**  
  Calculate expected logs, metrics, and events per day (GB or TB).
- **CPU & Memory:**  
  More cores and RAM improve indexing and search performance.
- **Storage:**
  - Use fast disk (SSD recommended) for hot/warm buckets.
  - Plan for retention and retention policies (hot, warm, cold, frozen buckets).
- **Network:**  
  Ensure low-latency, high-throughput connectivity between forwarders and indexers.

**Reference Sizing Calculator:**  
[Splunk Sizing Documentation](https://docs.splunk.com/Documentation/Splunk/latest/Capacity/Summaryofperformancerecommendations)

---

## Indexer Cluster Setup (Distributed Architecture)

### 1. Prepare Your Environment

- Provision at least 3 indexer nodes for fault tolerance.
- Set up a separate Cluster Master (Manager) node.
- (Optional) Have one or more dedicated Search Heads.

### 2. Install Splunk Enterprise

Install Splunk Enterprise on each node (indexers, master, search heads).  
[Download Splunk Enterprise](https://www.splunk.com/en_us/download/splunk-enterprise.html)

### 3. Configure the Cluster Master

- On the Cluster Master, edit or create `$SPLUNK_HOME/etc/system/local/server.conf`:
  ```ini
  [clustering]
  mode = master
  pass4SymmKey = your-cluster-key
  ```

### 4. Configure Peer (Indexer) Nodes

- On each indexer, configure as a cluster slave/peer in `server.conf`:
  ```ini
  [clustering]
  mode = slave
  master_uri = https://<master-host>:8089
  pass4SymmKey = your-cluster-key
  ```

### 5. Configure Search Heads

- On the search head, configure to search the cluster:
  ```ini
  [clustering]
  mode = searchhead
  master_uri = https://<master-host>:8089
  pass4SymmKey = your-cluster-key
  ```

### 6. Start and Validate

- Start/restart Splunk on all nodes:
  ```bash
  $SPLUNK_HOME/bin/splunk restart
  ```
- Validate cluster health and peer status using the Splunk Web UI or CLI.

---

## Best Practices

- **Use a unique `pass4SymmKey`** for security across your cluster.
- **Monitor indexer performance** (CPU, I/O, latency) with the [Monitoring Console](https://docs.splunk.com/Documentation/Splunk/latest/MonitoringConsole/MCoverview).
- **Apply retention and index policies** to manage storage and compliance.
- **Keep indexers on the same network segment** for minimum latency.
- **Regularly back up configuration and indexed data.**
- **Use dedicated hardware** for indexers in production environments.

---

## References

- [Splunk Indexer Clustering Overview](https://docs.splunk.com/Documentation/Splunk/latest/Indexer/Aboutclusters)
- [Splunk Enterprise Installation Guide](https://docs.splunk.com/Documentation/Splunk/latest/Installation/InstallSplunk)
- [Splunk Capacity Planning](https://docs.splunk.com/Documentation/Splunk/latest/Capacity/CapacityplanningforSplunkEnterprise)
- [Splunk Monitoring Console](https://docs.splunk.com/Documentation/Splunk/latest/MonitoringConsole/MCoverview)
