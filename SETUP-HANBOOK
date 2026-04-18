# FinSOC Lab – Professional Step-by-Step Handbook  
**Version 2.3 – April 2026**  
**Built for Lagos-based aspiring SOC Analysts**

## 1. Project Overview  
**FinSOC Lab** is a complete, production-grade virtual Security Operations Center (SOC) lab simulating a Nigerian fintech company’s Security Operations Center.  

It demonstrates full-stack practical skills that hiring managers look for in SOC Analyst and Junior SOC Engineer roles:

- Host-based monitoring (Wazuh agents) + network-based detection (Suricata NIDS/IPS)  
- Custom detection rules on both tools  
- Automated containment using Wazuh Active Response with expanded custom Python scripting  
- ELK Stack integration for advanced visualization and analytics  
- Incident correlation, NIST-based playbooks, and response  

**Target Hardware**: 16 GB RAM laptop (Core i7 recommended).

## 2. Hardware & Lab Requirements  
**VM Allocations (Total peak ~12–14 GB)**:  
- **Wazuh All-in-One Server** (Ubuntu 24.04 LTS): 8 GB RAM, 4–6 vCPU, 120 GB dynamic disk  
- **Windows Endpoint**: 4 GB RAM, 2 vCPU, 80 GB disk  
- **Linux Endpoint** (with Suricata): 3–4 GB RAM, 2 vCPU, 50 GB disk  
- **Attacker VM** (Kali Linux): 2–3 GB RAM, 2 vCPU, 60 GB disk  

**Hypervisor**: Oracle VirtualBox 7.x (latest version recommended)

## 3. Lab Architecture Overview  
(See `ARCHITECTURE.md` and `architecture-diagram.png`)

**Data Flow**:  
Kali Attacker → Suricata (Linux Endpoint) → Wazuh Agent → Wazuh Server → Custom Rules → Active Response (Python script) → Optional ELK Stack (Logstash → Kibana).

# 4. Step-by-Step Build Guide  

## Phase 1: Prepare Host and Create VMs

1. **Install Oracle VirtualBox**  
   Download and install the latest version of Oracle VirtualBox (7.x) from the official website: [virtualbox.org](https://www.virtualbox.org).

2. **Download Required ISOs**  
   - Ubuntu 24.04 LTS Desktop or Server ISO (for Wazuh Server and Linux Endpoint)  
   - Windows 11 ISO (for Windows Endpoint – use evaluation/trial version if needed)  
   - Kali Linux ISO (for Attacker VM)

3. **Create the Wazuh All-in-One Server VM (First)**  
   - Click **New** in VirtualBox.  
   - **Name**: `FinSOC-Wazuh-Server`  
   - **Type**: Linux → Ubuntu (64-bit)  
   - **Memory (RAM)**: 8192 MB (8 GB) – temporarily increase to 10 GB if installation is slow  
   - **Processors (CPU)**: 4–6 cores  
   - **Hard Disk**: Create a virtual hard disk now → Dynamically allocated → 120 GB  
   - **Network**: Start with **Host-only Adapter** (create one if none exists). Switch temporarily to Bridged for internet access during updates.  
   - Attach the Ubuntu ISO to the optical drive.  
   - Under **System** tab, enable **PAE/NX** and **Nested VT-x/AMD-V** if available.  
   - Start the VM and install Ubuntu:  
     - Choose **Minimal installation**.  
     - Create a user (e.g., `socadmin`).  
     - **Important**: Enable **OpenSSH server** during installation for easy access from your host.  
   - After installation, update the system:  
 
     sudo apt update && sudo apt upgrade -y
     sudo apt install curl net-tools -y
Note the VM’s IP address using ip addr show

4. **Create the Remaining Three VMs** with recommended allocations:

   **A. Windows Endpoint VM**  
   - **Name**: `FinSOC-Windows-Endpoint`  
   - **Type**: Microsoft Windows → Windows 11 (64-bit)  
   - **Memory**: 4096 MB (4 GB)  
   - **Processors**: 2 cores  
   - **Hard Disk**: Dynamically allocated → 80 GB  
   - **Network**: Host-only Adapter  
   - Attach the Windows 11 ISO and perform a normal installation.  
   - After installation, deploy the Wazuh agent from the Wazuh dashboard (Agents → Deploy new agent).

   **B. Linux Endpoint VM (with Suricata)**  
   - **Name**: `FinSOC-Linux-Endpoint`  
   - **Type**: Linux → Ubuntu (64-bit)  
   - **Memory**: 3072–4096 MB (3–4 GB)  
   - **Processors**: 2 cores  
   - **Hard Disk**: Dynamically allocated → 50 GB  
   - **Network**: Host-only Adapter  
   - Attach the Ubuntu ISO and install Ubuntu (enable **OpenSSH server** during installation).  
   - This VM will run **Suricata** + **Wazuh agent**.

   **C. Attacker VM (Kali Linux)**  
   - **Name**: `FinSOC-Kali-Attacker`  
   - **Type**: Linux → Debian (64-bit) or Other Linux  
   - **Memory**: 2048–3072 MB (2–3 GB)  
   - **Processors**: 2 cores  
   - **Hard Disk**: Dynamically allocated → 60 GB  
   - **Network**: Host-only Adapter  
   - Attach the Kali Linux ISO and install Kali.  
   - This VM will be used for safe attack simulations (e.g., `nmap`, `hydra`, etc.).

**Important Notes for All VMs**:
- Use **Host-only Adapter** for all VMs to keep the lab isolated and secure.
- Ensure all VMs are connected to the same Host-only network so they can communicate.
- Enable hardware virtualization (VT-x / AMD-V) in your laptop BIOS if not already enabled.
- After creating each VM, go to **Settings → Network** and confirm they are on the same adapter.
- Close unnecessary applications on your host laptop while running the lab to save RAM.

Once all four VMs are created and the Wazuh Server is installed and running, proceed to **Phase 2**.

## Phase 2: Install Wazuh All-in-One Server

Inside the **Wazuh Server VM** run:

 **Download the Wazuh installation assistant**:
 ```
 curl -sO [https://packages.wazuh.com/4.x/wazuh-install.sh](https://packages.wazuh.com/4.x/wazuh-install.sh)
 ```
**Run the installation script with the all-in-one flag**:
 ```
 sudo bash wazuh-install.sh -a
 ```
**After installation completes**:

The script will display the admin password at the end. Copy and save it securely — you will need it to log into the Wazuh dashboard.
Note the IP address of the Wazuh Server VM (run ip addr show if needed).

**Access the Wazuh Dashboard**:

- From your host laptop browser, open:
```
https:// https://192.168.56.10
- Accept the self-signed certificate warning (this is normal in a lab environment).
```  
- Log in with:
```  
Username: admin
Password: (the one generated during installation)
```

You should now see the Wazuh dashboard with 0 agents connected.
Next Step: Proceed to Phase 3 to deploy Wazuh agents on the Windows and Linux endpoints.
Troubleshooting Tips:

If the installation fails due to memory issues, temporarily increase the Wazuh VM RAM to 10 GB, then reduce it back after installation.
Make sure the VM has internet access during the installation (switch to Bridged network temporarily if needed).
The installation usually takes 10–30 minutes depending on your internet speed.

## Phase 3: Deploy Wazuh Agents on Endpoints

Once the server is ready, you must install the Wazuh agent on your endpoints to begin data collection.

#### 1. Windows Endpoint Deployment
1. Log in to the **Wazuh Dashboard** (`https://<wazuh-vm-ip>`).
2. Navigate to **Wazuh** -> **Agents** -> **Deploy new agent**.
3. Select the following options:
   - **Package type**: `Windows (MSI)`
   - **Wazuh server address**: Enter the static IP of your `FinSOC-Wazuh-Server`.
   - **Agent name**: `FinSOC-Win-01`
4. Copy the generated PowerShell command.
5. Open **PowerShell (Admin)** on your Windows VM and paste/run the command.
6. Start the agent:
   ```powershell
   NET START Wazuh

### 2. Linux Endpoint Deployment (Ubuntu)

On the **Linux Endpoint** VM, follow these steps to install and register the agent:

1. Log in to the **Wazuh Dashboard** and navigate to **Agents** > **Deploy new agent**.
2. Select the following configuration:
   - **Package type**: `Linux DEB`
   - **Architecture**: `amd64`
   - **Wazuh server address**: `<Your-Wazuh-Server-IP>`
   - **Agent name**: `FinSOC-Linux-01`
3. Copy the registration command provided by the dashboard.
4. Open the terminal on your **Linux Endpoint** and execute the copied command.
5. Once installed, run the following commands to enable and start the agent service:

    ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable wazuh-agent
   sudo systemctl start wazuh-agent
   ```

### 3. Verification

After completing the deployments, verify the connection status to ensure telemetry is flowing to the manager:

1. **Return to the Wazuh Dashboard**: Log in to the web interface.
2. **Navigate to the Agents tab**: Click on the **Wazuh** menu and select **Agents**.
3. **Confirm Connection**: Verify that both `FinSOC-Win-01` and `FinSOC-Linux-01` appear in the list with the status **Active** (indicated by a green checkmark).

---

> [!TIP]
> **Troubleshooting Connectivity**
> If an agent does not appear as **Active** within 2 minutes, check the logs for errors related to network connectivity or authentication:
>
> * **Linux**: 
>   ```bash
>   sudo tail -f /var/ossec/logs/ossec.log
>   ```
> * **Windows**: 
>   Check the log file located at: `C:\Program Files (x86)\ossec-agent\ossec.log`



## Phase 4: Integrate Suricata on Linux Endpoint

In this phase, you will install Suricata (Network Intrusion Detection System) on the **Linux Endpoint** and configure the Wazuh agent to forward its logs to the Wazuh Server.

### 1. Install Suricata
Run these commands on the **Linux Endpoint VM** to add the stable repository and install the package:

```bash
# Update system and add Suricata stable PPA
sudo apt update && sudo apt upgrade -y
sudo add-apt-repository ppa:oisf/suricata-stable -y
sudo apt update

# Install Suricata
sudo apt install suricata -y
```

### 2. Configure Suricata

You must edit the main configuration file to ensure Suricata is listening on the correct interface and monitoring the appropriate network range.

1.  **Open the configuration file**:
    ```bash
    sudo nano /etc/suricata/suricata.yaml
    ```

2.  **Update the following parameters**:

    * **HOME_NET**: Define your lab's internal subnet.
        * Locate the line `HOME_NET: "[192.168.0.0/16]"` and change it to your specific subnet (e.g., `192.168.56.0/24`).
    * **af-packet**: Assign Suricata to your active network interface.
        * Run `ip a` in a separate terminal to find your interface name (e.g., `eth0` or `enp0s3`).
        * Under the `af-packet` section in the YAML file, change the `interface: eth0` line to match your actual interface name.
    * **eve-log**: Ensure detailed JSON logging is enabled so Wazuh can ingest the data.
        * Confirm that under `outputs:`, the `- eve-log:` section has `enabled: yes` set.

3.  **Save and Exit**:
    * Press `CTRL + O`, then `Enter` to save.
    * Press `CTRL + X` to exit the editor.

---

> [!TIP]
> **Verify Syntax**
> You can verify your Suricata configuration for syntax errors before restarting by running:
> ```bash
> sudo suricata -T -c /etc/suricata/suricata.yaml -v
> ```


### 3. Connect Suricata to Wazuh Agent

To allow Wazuh to ingest network security events, you must modify the agent configuration to monitor the Suricata output file.

1.  **Open the Wazuh agent configuration**:
    ```bash
    sudo nano /var/ossec/etc/ossec.conf
    ```

2.  **Add the log monitoring block**:
    Scroll down to the `<ossec_config>` section and add the following XML block:

    ```xml
    <localfile>
      <log_format>json</log_format>
      <location>/var/log/suricata/eve.json</location>
    </localfile>
    ```

3.  **Save and Exit**:
    * Press `CTRL + O`, then `Enter` to save.
    * Press `CTRL + X` to exit.


### 4. Restart Services

To apply the new configurations and begin monitoring, restart both the Suricata engine and the Wazuh agent.

1.  **Execute the restart command**:
    ```bash
    sudo systemctl restart suricata wazuh-agent
    ```

2.  **Verify Service Status (Optional)**:
    ```bash
    sudo systemctl status suricata wazuh-agent
    ```

---

> [!NOTE]
> **Log Validation**
> After restarting, you can verify that Suricata is successfully generating logs by checking the `eve.json` file:
> ```bash
> sudo tail -f /var/log/suricata/eve.json
> ```
> If the file is being populated with JSON data, the Wazuh agent will automatically begin ingesting those network alerts and forwarding them to your dashboard.




## Phase 5: Custom Suricata Rules

In this phase, you will implement custom detection rules to identify specific malicious traffic patterns within your network.

### 1. Create the Custom Rules File
On the **Linux Endpoint VM**, create the file and populate it with your detection rules:

```bash
# Create the rules file
sudo nano /etc/suricata/rules/project-alert.rules
```

Paste your rules from `detection-rules/suricata-project-alert.rules` into this file.

**Example rule:**

```text
alert icmp $EXTERNAL_NET any -> $HOME_NET any (msg:"ICMP Ping Detected"; sid:1000001; rev:1;)
```
> [!TIP]
> **Understanding the Rule Syntax**
>
> * **alert**: The action to take when the rule is matched.
> * **icmp**: The protocol to monitor.
> * **sid**: The "Signature ID," which must be unique for every rule you create.


### 2. Update Suricata Configuration

You must tell Suricata to load this specific rule file so it can start processing your custom alerts.

1.  **Open the configuration file**:
    ```bash
    sudo nano /etc/suricata/suricata.yaml
    ```

2.  **Add the file to the rule-files list**:
    Scroll down to the `rule-files:` section and add the path to your new file:

    ```yaml
    rule-files:
      - suricata.rules
      - /etc/suricata/rules/project-alert.rules
    ```

3.  **Save and Exit**:
    * Press `CTRL + O`, then `Enter` to save.
    * Press `CTRL + X` to exit.


### 3. Restart and Test

Apply the changes and test the detection from your Kali VM to ensure the custom rule is functioning correctly.

1.  **Restart Suricata**:
    ```bash
    sudo systemctl restart suricata
    ```

2.  **Test from Kali Attacker VM**:
    Perform a simple ping or scan against the Linux Endpoint IP to trigger the rule:
    ```bash
    ping <Linux-Endpoint-IP>
    ```

3.  **Verify Detection**:
    Check the logs on the **Linux Endpoint** to see if the custom rule triggered:
    ```bash
    sudo tail -f /var/log/suricata/eve.json | grep "ICMP Ping Detected"
    ```

---

> [!TIP]
> **Dashboard Check**
> Once the rule triggers in the local `eve.json` file, check your **Wazuh Dashboard** under the **Security Events** tab. You should see the Suricata alert appear there, as the Wazuh agent is now forwarding these JSON events.


## Phase 6: Wazuh Custom Rules

In this phase, you will configure the Wazuh Manager to recognize specific log patterns and assign them risk levels by updating the local ruleset.

### 1. Update the Local Rules File
On the **Wazuh Server VM**, you need to append your custom rules to the default local ruleset.

1.  **Open the local rules file**:
    ```bash
    sudo nano /var/ossec/etc/rules/local_rules.xml
    ```

2.  **Add your custom rules**:
    Copy the content from `detection-rules/wazuh-local_rules.xml` and paste it inside the `<group name="local,">` tags.

3.  **Save and Exit**:
    * Press `CTRL + O`, then `Enter` to save.
    * Press `CTRL + X` to exit.

### 2. Apply and Restart
The manager must be restarted to load the new XML configuration into memory.

1.  **Restart the Wazuh Manager**:
    ```bash
    sudo systemctl restart wazuh-manager
    ```

### 3. Test with Wazuh-Logtest
Before relying on the rules in production, use the built-in log testing tool to verify that your rules trigger as expected.

1.  **Run the logtest utility**:
    ```bash
    sudo /var/ossec/bin/wazuh-logtest
    ```

2.  **Perform the test**:
    * Paste a sample log string (e.g., a failed SSH login or a specific Suricata JSON event) into the prompt.
    * **Verify Output**: The tool will show you which phase of the analysis matched and which **Rule ID** was triggered.

---

> [!NOTE]
> **Rule Syntax Validation**
> If the manager fails to restart, check the configuration for syntax errors (like unclosed XML tags) using:
> ```bash
> sudo /var/ossec/bin/wazuh-analysisd -t
> ```



## Phase 7: Wazuh Active Response – Expanded Scripting

In this phase, you will implement an automated response script that enables Wazuh to dynamically block IP addresses based on Suricata alerts.

### 1. Deploy the Active Response Script

On the **Wazuh Server** (or the Linux Endpoint if executing locally), move the custom Python script to the production directory and set the appropriate security permissions:

```bash
# Copy the script from your project directory
sudo cp active-response/suricata-block-ip.py /var/ossec/active-response/bin/

# Set necessary permissions and ownership
sudo chmod 750 /var/ossec/active-response/bin/suricata-block-ip.py
sudo chown root:wazuh /var/ossec/active-response/bin/suricata-block-ip.py
```

### 2. Configure the Command Block

Open the Wazuh Manager configuration file to define the script as a recognized command that the system can execute when triggered.

1.  **Open the configuration file**:
    ```bash
    sudo nano /var/ossec/etc/ossec.conf
    ```

2.  **Add the `<command>` block**:
    Scroll down to the commands section and insert the following XML configuration:

    ```xml
    <command>
      <name>suricata-block-ip</name>
      <executable>suricata-block-ip.py</executable>
      <timeout_allowed>yes</timeout_allowed>
    </command>
    ```

3.  **Save and Exit**:
    * Press `CTRL + O`, then `Enter` to save.
    * Press `CTRL + X` to exit.

### 3. Configure the Active-Response Block

In the same `ossec.conf` file, define the trigger conditions and the escalating timeout for repeat offenders. This block links the previous command to specific rule alerts.

```xml
<active-response>
  <command>suricata-block-ip</command>
  <location>local</location>
  <rules_id>100001</rules_id> 
  <timeout>600</timeout>
  <repeated_offenders>30,60,120</repeated_offenders>
</active-response>
```
> [!IMPORTANT]
> **Understanding the Parameters**
>
> * **rules_id**: The specific Wazuh rule ID that will trigger this response.
> * **timeout**: The duration (in seconds) the IP will be blocked initially.
> * **repeated_offenders**: A comma-separated list of increasing timeout values (in minutes) for IPs that trigger the rule multiple times.


### 4. Restart Services and Test

Apply the changes and verify the automation to ensure the active response system is correctly intercepting threats.

1.  **Restart the Wazuh Manager**:
    ```bash
    sudo systemctl restart wazuh-manager
    ```

2.  **Test the Trigger**:
    From your **Kali VM**, generate network traffic (such as a scan or specific exploit) that triggers the defined Suricata rule.

3.  **Verify the Block**:
    Check the firewall rules on the **target endpoint** to confirm the attacker's IP has been dynamically added to the drop list:
    ```bash
    # On the Linux Endpoint
    sudo iptables -L -n
    ```

---

> [!IMPORTANT]
> **Repeated Offenders**
> The `repeated_offenders` attribute ensures that if the same IP triggers the alert multiple times, the block duration increases (e.g., 30m, 60m, 120m) to mitigate persistent threats effectively.



## Phase 8: ELK Stack Integration

In this final phase, you will extend your monitoring capabilities by integrating the ELK (Elasticsearch, Logstash, Kibana) stack to provide advanced visualization and long-term log retention.

### 1. Install Logstash
On the **Wazuh Server** (or a dedicated lightweight VM), install the Logstash engine to handle the data pipeline:

```bash
# Import the Elastic Public Signing Key
wget -qO - [https://artifacts.elastic.co/GPG-KEY-elasticsearch](https://artifacts.elastic.co/GPG-KEY-elasticsearch) | sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg

# Add the Elastic repository
echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] [https://artifacts.elastic.co/packages/8.x/apt](https://artifacts.elastic.co/packages/8.x/apt) stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list

# Install Logstash
sudo apt-get update && sudo apt-get install logstash
```

### 2. Configure the Logstash Pipeline

Direct Wazuh alerts into the ELK pipeline by creating a configuration file that defines how logs are ingested and where they are sent.

1.  **Create the pipeline file**:
    ```bash
    sudo nano /etc/logstash/conf.d/wazuh-elk.conf
    ```

2.  **Apply Configuration**:
    Copy the content from `elk-integration/logstash-pipeline.conf` into this file. It should look similar to this:

    ```json
    input {
      file {
        path => "/var/ossec/logs/alerts/alerts.json"
        codec => "json"
        type => "wazuh"
      }
    }

    output {
      elasticsearch {
        hosts => ["http://localhost:9200"]
        index => "wazuh-alerts-%{+YYYY.MM.dd}"
      }
    }
    ```

3.  **Save and Exit**:
    * Press `CTRL + O`, then `Enter` to save.
    * Press `CTRL + X` to exit.

### 3. Enable and Start Services

Apply the new pipeline configuration and ensure Logstash is configured to start automatically on system boot.

```bash
# Enable and start Logstash
sudo systemctl enable logstash
sudo systemctl start logstash

# Verify the service is running
sudo systemctl status logstash
```

### 4. Create Correlation Dashboards

1.  **Access the Kibana interface**:
    Open your web browser and navigate to `http://<Kibana-IP>:5601`.

2.  **Define Index Patterns**:
    Navigate to **Stack Management > Index Patterns** and create an index pattern for `wazuh-alerts-*` to allow Kibana to retrieve the data from Elasticsearch.

3.  **Build Visualizations**:
    Go to the **Dashboard** section to create visualizations that correlate Suricata network alerts with Wazuh system events for a unified security posture.

---

> [!TIP]
> **Data Flow**
> The integration follows this path: 
> **Suricata (Log)** -> **Wazuh Agent** -> **Wazuh Manager (Alert)** -> **Logstash** -> **Elasticsearch** -> **Kibana**. 
>
> This ensures that every network threat detected is both acted upon by Wazuh's active response and visualized within the ELK stack.



## Phase 9: Suricata IPS Mode (Optional)

In this phase, you will transition Suricata from a passive IDS to an active **Intrusion Prevention System (IPS)**, allowing it to drop malicious packets in real-time using `nfqueue`.

### 1. Configure Iptables to Redirect Traffic
You must direct network traffic to the Netfilter queue where Suricata can inspect and decide whether to drop it.

```bash
# Redirect all incoming traffic to NFQUEUE
sudo iptables -I INPUT -j NFQUEUE --queue-bypass

# Redirect all outgoing traffic to NFQUEUE
sudo iptables -I OUTPUT -j NFQUEUE --queue-bypass
```

### 2. Update Suricata Configuration for IPS

Modify the `suricata.yaml` file to enable the `nfqueue` monitoring mode, transitioning Suricata from passive detection to active prevention.

1.  **Open the configuration file**:
    ```bash
    sudo nano /etc/suricata/suricata.yaml
    ```

2.  **Enable NFQUEUE**:
    Find the `nfqueue` section and ensure it is configured with safety features enabled:

    ```yaml
    nfqueue:
      mode: accept
      repeat-mark: 1
      repeat-mask: 1
      route-queue: 0
      fail-open: yes # Crucial for safety: traffic flows if Suricata crashes
    ```

3.  **Save and Exit**:
    * Press `CTRL + O`, then `Enter` to save.
    * Press `CTRL + X` to exit.

### 3. Set Rules to "Drop"

For the IPS to take action, your rules must be updated from `alert` (which only logs the event) to `drop` (which actively stops the packet).

1.  **Modify your custom rules file**:
    ```bash
    sudo nano /etc/suricata/rules/project-alert.rules
    ```

2.  **Change rule action**:
    Update specific critical rules to drop traffic. Example of a drop rule for ICMP traffic:
    ```plaintext
    drop icmp $EXTERNAL_NET any -> $HOME_NET any (msg:"ICMP Ping Blocked"; sid:1000001; rev:2;)
    ```

### 4. Restart Suricata

Apply the new IPS configuration and ruleset by restarting the Suricata service:

```bash
sudo systemctl restart suricata
```
> [!CAUTION]
> **Safety First**
>
> Always set `fail-open: yes`. This configuration prevents your network from losing connectivity entirely if the Suricata process hangs or fails. Ensure you thoroughly test this configuration in a lab environment before deploying it to a production network.


## Phase 10: Attack Simulations, Testing & Documentation

In this final phase, you will validate the entire security stack by simulating real-world attacks, verifying the automated response chain, and documenting the results according to standard incident response protocols.

### 1. Execute Attack Simulations
Use your **Kali Attacker VM** to generate malicious traffic and system events.

* **Network Scanning**:
    ```bash
    # Run a stealthy SYN scan to trigger Suricata/Wazuh alerts
    sudo nmap -sS -A <Target-IP>
    ```

* **Brute Force Simulation**:
    ```bash
    # Attempt an SSH brute force attack
    hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://<Target-IP>
    ```

* **Web Vulnerability Scanning**:
    ```bash
    # Scan for common web vulnerabilities (if a web server is deployed)
    nikto -h http://<Target-IP>
    ```

### 2. Verify the Defense Chain
Monitor the Wazuh and ELK dashboards to ensure the detection and response mechanisms are operating correctly.

1.  **Detection**: Verify that Suricata or Wazuh successfully identified the malicious activity.
2.  **Correlation**: Check the **Kibana** or **Wazuh Dashboard** to see if the alerts are being aggregated and visualized properly.
3.  **Active Response**: Verify that the attacker’s IP has been blocked by checking the target's firewall:
    ```bash
    sudo iptables -L -n
    ```

### 3. Document Incidents
Follow the predefined templates in your project repository to record the details of the simulated breaches.

* **Navigate to the documentation directory**:
    ```bash
    cd incident-playbooks/
    ```
* **Complete the Playbooks**: Fill out the response steps, mitigation strategies, and post-mortem analysis for each attack type (e.g., `ssh-bruteforce-playbook.md`, `network-scan-playbook.md`).

---

> [!NOTE]
> **Validation Check**
> A successful test is confirmed when a scan from the Kali VM results in an automated block, an alert in the Wazuh manager, and a corresponding entry in the ELK dashboard.




# Troubleshooting Tips

If you encounter issues during deployment or testing, use the following steps to diagnose and optimize your environment.

### 1. Monitor Key Logs
Check the following logs on the **Wazuh Server** and **Endpoints** to identify errors in detection or response:

* **Wazuh Manager/Agent Logs**:
    ```bash
    tail -f /var/ossec/logs/ossec.log
    ```

* **Suricata Event Logs**:
    ```bash
    tail -f /var/log/suricata/eve.json
    ```

* **Active Response Execution Logs**:
    ```bash
    tail -f /var/ossec/logs/active-responses.log
    ```

### 2. Prevent Self-Lockout

To avoid accidentally blocking your own management or attack machine during testing, whitelist your **Kali Linux IP** in the Wazuh global configuration:

1.  **Open ossec.conf**:
    ```bash
    sudo nano /var/ossec/etc/ossec.conf
    ```

2.  **Add your IP to the `<global>` section**:
    Locate the `<global>` block and add your specific IP address to the whitelist to ensure the Active Response module ignores traffic from your machine.

    ```xml
    <global>
      <white_list>127.0.0.1</white_list>
      <white_list>YOUR_KALI_IP</white_list>
    </global>
    ```

3.  **Restart the manager**:
    Apply the changes by restarting the Wazuh service:
    ```bash
    sudo systemctl restart wazuh-manager
    ```

### 3. Resource Management

Running a full SOC lab can be hardware-intensive. Follow these tips to maintain stability and prevent host system crashes.

* **Host Performance**: Close unnecessary browser tabs and background applications on your physical host while the VMs are running to free up CPU cycles and physical RAM.
* **VM RAM Allocation**:
    * If the installation process is hanging or extremely slow, temporarily increase the **Wazuh VM RAM to 10 GB**.
    * Once the installation is complete, you can scale it back down to **6 GB - 8 GB** for regular operation.

---

> [!TIP]
> **Proactive Monitoring**
> Use the `top` or `htop` command on your Linux VMs to monitor real-time CPU and RAM consumption if the system becomes unresponsive.


    
