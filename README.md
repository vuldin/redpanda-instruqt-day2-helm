# Day2 operations with the Redpanda helm chart

This is the repo for the Insruqt-based Redpanda day2 operations masterclass for helm-based deployments.

There are 4 scenarios focused on the following tasks:

1. [Overview guide](./01-day2-helm-overview/assignment.md)
2. [Node replacement (and commissioning brokers)](./02-day2-helm-node-replacement/assignment.md)
3. [Rolling Upgrades](./03-day2-helm-rolling-upgrade/assignment.md)
4. [Monitoring and Alerts](./04-day2-helm-monitoring-alerting/assignment.md)

Scenarios 1 through 3 make use of the Kubernetes cluster, and then scenario 4 spins up an environment with docker compose.

You will come away from this session knowing how to:

- Spin up an Kubernetes cluster and deploy Redpanda with Console, TLS, SASL, Tiered storage, and external access via LoadBalancer services
- Create additional node pools and migrate Redpanda over to the new node pool
- Use RPK to verify cluster status and health
- Deploy Prometheus and Grafana using their helm charts, and then install Grafana charts to view Redpanda metrics
- Configure alerting
- Add and remove brokers
- Update the cluster to a new version of Redpanda

We will answer your questions throughout this interactive session.

