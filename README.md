

```markdown
# ActiveDefense SOC Sandbox

An enterprise-grade, security-first Security Operations Center (SOC) sandbox engineered to simulate real-world cyber attacks, automate telemetry aggregation, and execute real-time incident mitigation. 

This repository serves as complete technical documentation for building, breaking, and tuning an automated Intrusion Detection and Prevention System (IDS/IPS) inside an isolated virtual network architecture.

---

## 1. Network Architecture & Topology

The entire infrastructure is isolated inside an Oracle VirtualBox dedicated **NAT Network** interface running on the subnet `10.0.2.0/24`. This setup simulates an enterprise DMZ while protecting the host machine from live-fire attack tools.

```text
       [ Kali Linux ] (Attacker / Analyst Dashboard)
              │ (10.0.2.x)
              ▼
   ─── VirtualBox NAT Network (10.0.2.0/24) ───
              ▲                               ▲
              │ (10.0.2.3)                    │ (10.0.2.6)
      [ Ubuntu Server ]               [ Ubuntu Client ]
   (Wazuh SIEM Master Stack)       (Target Production Victim)

```

### Virtual Machine Inventory

1. **Attacker / Analyst Node (Kali Linux):** Acts as both the adversarial threat source (generating brute-force traffic) and the security analyst terminal (accessing the web UI dashboard).
2. **SIEM Master Node (Ubuntu Server 24.04 LTS | `10.0.2.3`):** Orchestrates the decoupled core data and indexing engines using an isolated microservices container architecture.
3. **Victim Endpoint (Ubuntu Minimal | `10.0.2.6`):** Simulates a live production server. Runs a lightweight logging agent that ships telemetry upstream to the master node over a secure TCP handshake.

---

## 2. Software & Dependencies Matrix

| Software / Component | Context & Function within Lab |
| --- | --- |
| **Wazuh Manager** | The behavioral analysis engine. Validates incoming telemetry logs against structural security decoders and custom rulesets. |
| **Wazuh Indexer** | A high-performance OpenSearch-based document storage engine. Parses, structures, and indexes raw security alerts into queryable databases. |
| **Wazuh Dashboard** | The visual analytics web portal. Provides data discovery tables, network trends, and cryptographic event tracking. |
| **Wazuh Agent** | Lightweight background daemon deployed on endpoints. Monitors memory, process trees, and auth logs (`/var/log/auth.log`). |
| **Docker Engine & Compose** | Container runtime engine. Orchestrates the SIEM stack into distinct, decoupled, scalable network layers. |
| **Hydra** | Network logon brute-forcing engine used to simulate automated multi-threaded credential stuffing against SSH ports. |
| **iptables** | Linux kernel-level packet filter. Used as the enforcement arm for our automated active response scripts to drop malicious packets. |

---

## 3. Installation & Engineering Workflow

### Phase 1: Virtual Network Provisioning

1. Created a dedicated global **NAT Network** in VirtualBox named `SecOpsNet` assigned to the IP space `10.0.2.0/24` with DHCP capabilities enabled.
2. Provisioned 3 virtual endpoints and modified their respective Network Adapter settings from default `NAT` to `NAT Network (SecOpsNet)`.

### Phase 2: Deploying the Decoupled SIEM Stack

1. Updated system repositories and provisioned the Docker runtime environment on the Ubuntu Master server.
2. Cloned the single-node deployment manifest from the official Wazuh repository organization.
3. Orchestrated and initialized the core infrastructure microservices running detached in the background:
```bash
sudo docker-compose up -d

```



### Phase 3: Telemetry Agent Deployment

1. Accessed the master endpoint configuration deployment interface to generate an isolated, cryptographically signed installation command string.
2. Deployed the agent binary straight onto the Victim machine, linked it explicitly back to the master controller IP (`10.0.2.3`), and activated the daemon:
```bash
sudo systemctl daemon-reload

```



### Phase 4: Transitioning from Passive Detection (IDS) to Prevention (IPS)

Configured the automated enforcement mechanisms inside the master telemetry control engine (`ossec.conf`). Consolidated redundant config structures into a unified well-formed schema to trap malicious automation behaviors:

```xml
<ossec_config>
  <command>
    <name>firewall-drop</name>
    <executable>firewall-drop</executable>
    <timeout_allowed>yes</timeout_allowed>
  </command>

  <active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <rules_id>5712</rules_id>
    <timeout>180</timeout>    
  </active-response>
</ossec_config>

```

---

## 4. Incident Log & Real-World Troubleshooting Documentation

### Incident 1: Frontend App Navigation Loss (UI State Loop)

* **Symptom:** The OpenSearch analytics engine threw an unhandled Promise rejection warning: `ScopedHistory instance has fell out of navigation scope for basePath: /app/data-explorer`. The visual dashboard froze.
* **Root Cause Analysis:** The web browser's state router dropped sync with the backend application routing state after rapid query changes. The underlying storage engines were running unaffected.
* **Resolution:** Executed a clean session browser cache bypass (`Ctrl + Shift + R`). For persistent errors, isolated the frontend microservice and safely forced a graceful reboot without dropping live alert queues:
```bash
sudo docker-compose restart wazuh.dashboard

```



### Incident 2: Storage Exhaustion & Watermark Database Freeze (100% Use)

* **Symptom:** Bringing up services failed with the error: `Cannot start service wazuh.dashboard: no space left on device`. The indexer locked into an immutable read-only mode.
* **Root Cause Analysis:** High-volume adversarial simulations generated large telemetry log footprints that quickly filled up the 28GB virtual storage allocation inside `/var/lib/docker/volumes`. Once storage utilization hit the 95% threshold, the high disk watermark lock tripped to prevent data corruption.
* **Resolution:** Cleared system packages (`sudo apt clean`), removed orphaned docker caching structures, and safely executed a controlled purge and re-indexing cycle to reclaim physical storage space:
```bash
sudo docker-compose down -v
sudo docker system prune -a --volumes -f
sudo docker-compose up -d

```



### Incident 3: Overlapping XML Schemas (Manager Fail-to-Boot)

* **Symptom:** The Wazuh backend engine encountered structural verification failures and refused to initialize after adding active defense components.
* **Root Cause Analysis:** The master configuration file contained multiple overlapping `<ossec_config>` parent blocks, violating strict well-formed XML parsing constraints.
* **Resolution:** Consolidated all disjoint settings blocks, independent parameters, and active-response directives cleanly into a singular, cohesive `<ossec_config>` system hierarchy block.

---

## 5. Proactive Risk Assessments: Vulnerability Remediation

Using the built-in global National Vulnerability Database (NVD) mappings, the lab successfully ran automated internal audits across the software inventory layers.

### Initial Triage Results

* **3 High-Severity Vulnerabilities** discovered in target base files.
* **Key Risks Isolated:** * `CVE-2023-50782` (Flaw within the Python cryptography encryption handling library).
* `CVE-2024-0727` (Null-pointer parsing error in OpenSSL handling certificate validation, leading to remote Denial of Service).



### Remediation Action Executed

Surgically targeted the vulnerable libraries on the victim server to lower the overall system attack surface without causing general downtime:

```bash
sudo apt update && sudo apt install --only-upgrade python3-cryptography openssl coreutils -y
sudo systemctl restart wazuh-agent

```

* **Result:** The vulnerability counts dropped to zero on subsequent automated checks.

---

## 6. Project Gaps & Future Roadmap

To move this system closer to an enterprise-grade production infrastructure environment, the following components are scheduled for integration:

* **File Integrity Monitoring (FIM) Real-time Hardening:** Configure specific system file watches on `/etc/passwd` and directory assets on local web servers to detect and log unauthorized system parameter shifts or line modifications.
* **Syslog Forwarding Integration:** Open network sockets (UDP port 514) on the manager to intake telemetry from bare-metal edge routers and managed switches.
* **Automated Log Rotation Architecture:** Set up aggressive index pruning policies within OpenSearch tools to maintain disk utilization under 70% automatically, preventing future storage exhaustion locks.

```

```
