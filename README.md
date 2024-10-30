# ELK Stack Installation Guide on 3 Nodes

This guide provides a step-by-step installation of the ELK Stack (Elasticsearch, Logstash, and Kibana) on a three-node setup. Each node hosts Elasticsearch, while Kibana and Logstash are set up on a central node (192.168.56.101) to manage data ingestion and visualization.

## Overview

The configuration enables distributed Elasticsearch services while centralizing Kibana and Logstash, ensuring a scalable and robust data pipeline and analytics environment.

## Architecture

- **Node 1 (192.168.56.101)**: Hosts Elasticsearch, Kibana, and Logstash
- **Node 2 (192.168.56.102)**: Hosts Elasticsearch
- **Node 3 (192.168.56.103)**: Hosts Elasticsearch

## Prerequisites

- **Java**: Ensure all nodes have Java installed.
- **Hosts File Configuration**: Map all nodes for smooth inter-node communication.
- **Firewall Rules**: Open necessary ports (9200 for Elasticsearch HTTP, 9300 for Elasticsearch transport, 5601 for Kibana, and 5044 for Logstash Beats).
- **Clock Synchronization**: Use `ntpd` or `chronyd` to synchronize the clock on all nodes, preventing timing issues in the cluster.

## Installation Guide

This guide includes the following setup steps:

1. **Elasticsearch Installation**: Steps for each node to install and configure Elasticsearch, create data directories, set up a cluster, and enable authentication with TLS.
2. **Kibana Installation**: Instructions to install Kibana on Node 1, configure security settings, and connect to Elasticsearch.
3. **Logstash Installation**: Detailed steps for installing Logstash on Node 1, configuring it to minimize log verbosity, and setting up Oracle Instant Client if using JDBC for database ingestion.
4. **LVM Setup (Optional)**: Guidelines to configure LVM on any node needing extra storage, format it as XFS, and mount it persistently for Elasticsearch data.
5. **Service Enablement**: Ensuring services are set to start automatically on boot for all nodes.

## Detailed Setup Steps

For a comprehensive setup, refer to the [ELK_Installation_Guide.md](ELK_Installation_Guide.md) document, which includes configurations, commands, and step-by-step instructions for each component.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributions

Contributions are welcome! Feel free to submit a pull request or open an issue if you have suggestions or find any issues.

---

By following this guide, you’ll set up a resilient and distributed ELK Stack, with centralized visualization and logging capabilities on a dedicated primary node. Perfect for scalable, distributed log and data analysis.