# VPC-Firewall-Configuration-Flow-Log-Analysis-and-Security-Remediation
Here is a complete, production-ready `README.md` file tailored specifically for a repository focused on **VPC Firewall Configuration, Flow Log Analysis, and Security Remediation** .

---

```markdown
# GCP VPC Traffic Control & Flow Log Analysis

## Overview
This project demonstrates the implementation of Google Cloud Platform (GCP) Virtual Private Cloud (VPC) firewall controls, traffic monitoring, and network security audit procedures. 

The objective of this environment is to test, analyze, and secure inbound network traffic to an Apache web server instance hosted within a custom-mode VPC network. It walks through configuring allow and deny rules, capturing network logs using VPC Flow Logs, and analyzing connection payloads within GCP Logs Explorer to verify traffic filtering controls.

---

## Architecture & Initial State

* **Network:** `vpc-net` (Custom-mode VPC network)
* **Subnet:** `vpc-subnet` (Configured with active VPC Flow Logs)
* **Target Workload:** `web-server` (Compute Engine VM running Apache Web Server)
* **Network Tag:** `http-server`

---

## Execution Steps

### Task 1: Create an Inbound Allow Rule
Create a firewall rule to permit initial HTTP and SSH traffic targeted directly at the web server via custom network tags.

```bash
gcloud compute firewall-rules create allow-http-ssh \
    --network=vpc-net \
    --action=ALLOW \
    --direction=INGRESS \
    --rules=tcp:80,tcp:22 \
    --target-tags=http-server \
    --source-ranges=0.0.0.0/0 \
    --enable-logging

```

* **Purpose:** Allows inbound HTTP (Port 80) and SSH (Port 22) connections to instances bearing the `http-server` target tag.

---

### Task 2: Generate & Capture Network Traffic

1. Retrieve the public IP address of the `web-server` VM instance via the Compute Engine console.
2. In a web browser, navigate to the external IP address:
```text
http://<WEB_SERVER_EXTERNAL_IP>/

```


3. Determine your current local public IPv4 address using an IP lookup tool (e.g., `whatismyip.com`).

---

### Task 3: Analyze VPC Flow Logs in Logs Explorer

1. Navigate to **Logs Explorer** in the GCP Console.
2. Filter logs for subnetwork traffic by selecting **Subnetwork** under Resource Type and choosing `compute.googleapis.com/vpc_flows` under Log Name.
3. Run the following structured query in the Query Builder (replacing `YOUR_LOCAL_IP` with your public IPv4 address):

```kql
resource.type="gce_subnetwork"
log_name="projects/YOUR_PROJECT_ID/logs/compute.googleapis.com%2Fvpc_flows"
jsonPayload.connection.src_ip="YOUR_LOCAL_IP"

```

#### Key Payload Fields Identified:

* `jsonPayload.connection.dest_ip`: Internal IP address of the target web server instance (`10.1.3.2`).
* `jsonPayload.connection.dest_port`: Destination port (`80` for HTTP).
* `jsonPayload.connection.protocol`: Transport layer protocol (`6` representing TCP).
* `jsonPayload.connection.src_ip`: Client source IP address.
* `jsonPayload.connection.src_port`: Ephemeral source port assigned to the client request.

---

### Task 4: Implement Inbound Deny Rule

Create a higher-priority or specific firewall rule to block incoming HTTP requests to the target web server.

```bash
gcloud compute firewall-rules create deny-http \
    --network=vpc-net \
    --action=DENY \
    --direction=INGRESS \
    --rules=tcp:80 \
    --target-tags=http-server \
    --source-ranges=0.0.0.0/0 \
    --enable-logging

```

* **Purpose:** Explicitly drops inbound HTTP traffic on port 80 aimed at instances tagged `http-server`.

---

## Verification & Key Takeaways

1. **Traffic Blocking:** Upon enforcing the `deny-http` rule, subsequent HTTP connection attempts to the external IP timeout, confirming active packet filtering.
2. **Log Audit:** Reviewing VPC Flow Logs confirms traffic state transitions from allowed packets to evaluated drops, demonstrating proper rule evaluation order within GCP VPC security configurations.

```

```
