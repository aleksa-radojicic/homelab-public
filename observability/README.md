# Observability
Centralized metrics, logs and alerting for the homelab and additional Tailnet devices.

The stack uses Alloy as its telemetry collection and forwarding layer, Prometheus for metrics storage, Loki for log aggregation, Grafana for visualization and Alertmanager for alert routing. Notifications are delivered through ntfy.

## Architecture
![Alt Architecture](./diagram.png)

The observability stack runs on the homelab server. Alloy collects telemetry from the host and its services and forwards metrics to Prometheus and logs to Loki. Additional Tailnet devices can run Alloy and forward their telemetry to the same central instances (push-model). Grafana visualizes the collected data, while Prometheus sends firing alerts to Alertmanager. Alertmanager forwards notifications to ntfy through `alertmanager-ntfy-bridge`.

View the code for the [diagram](./diagram.md).

## Data Collection
Alloy collects:
- **Host metrics** (Alloy's `node_exporter` integration)
- **Container metrics** (Alloy's cAdvisor integration)
- **Service availability** (Blackbox Exporter)
- **Hardware metrics** (`smartctl_exporter`)
- **Systemd metrics** (`systemd_exporter`)
- **DNS metrics** (`pihole-exporter`)
- **Caddy metrics**
- **Container logs**
- **journald logs**
- **File-based logs** from `/var/log/`

## Authentication
Observability services are accessed through the homelab reverse proxy and protected by Authelia.

Service accounts (e.g. 'metrics-pusher') are used for machine-to-machine access where required, such as metric and log ingestion.

## Usage
Install the required Ansible collections and prepare the environment:
```sh
ansible-galaxy collection install -r ansible/requirements.yaml
cp .env.example .env
vim .env
```

Set up and configure exporters:
```sh
cd ansible
ansible-playbook --ask-become-pass exporters.yaml
```

Set up and configure Alloy:
```sh
cd ansible
ansible-playbook --ask-become-pass alloy.yaml
```

Deploy observability stack:
```sh
docker network create homelab-net
docker compose up -d
```
