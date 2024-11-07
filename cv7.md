# RouterOS Configuration Tutorial for Exercise 9

## 1. Reset Router Configuration

1. Open **WinBox** or access the **RouterOS** device.
2. Go to **System -> Reset Configuration**.
3. Check the box **No Default Configuration** to start from a clean state.
4. Click **Reset Configuration** to confirm. The router will reboot.

---

## 2. Basic Configuration (Bridge, LAN, and WAN Setup)

### 2.1 Create a Bridge for LAN (eth2, eth3, eth4)

1. Go to **Interfaces -> Bridge**.
2. Click **Add** (`+`) and name it `bridge-lan`.
3. Go to the **Ports** tab.
4. Add **ether2**, **ether3**, and **ether4** to the `bridge-lan`:
   - For each, click **Add** (`+`), set **Interface** to the respective port (e.g., `ether2`), and set **Bridge** to `bridge-lan`.
   - Click **OK** after adding each interface.

**Verification:**

- Go to **Interfaces** and confirm that **bridge-lan** is listed.
- Under **Interfaces -> Bridge -> Ports**, confirm that **ether2**, **ether3**, and **ether4** are added to `bridge-lan`.

### 2.2 Assign IP Address to the Bridge

1. Go to **IP -> Addresses**.
2. Click **Add** (`+`).
3. In the **Address** field, enter `10.0.X.1/24` (replace `X` with your assigned number).
4. Set the **Interface** to `bridge-lan`.
5. Click **OK**.

**Verification:**

- Under **IP -> Addresses**, confirm that `10.0.X.1/24` is assigned to `bridge-lan`.

### 2.3 Set up DHCP Server on the Bridge

1. Go to **IP -> DHCP Server -> DHCP Setup**.
2. Select **bridge-lan** as the interface.
3. Configure:
   - **DHCP Address Space**: `10.0.X.0/24`.
   - **Gateway for DHCP Network**: `10.0.X.1`.
   - **Address to Give Out**: `10.0.X.2 - 10.0.X.254`.
4. Click **Next** until the setup is complete.

**Verification:**

- Under **IP -> DHCP Server**, confirm that a DHCP server is running on `bridge-lan`.
- Under **IP -> Pool**, confirm that the pool `dhcp_pool1` is set correctly.

### 2.4 Set up DHCP Client on WAN (ether1)

1. Go to **IP -> DHCP Client**.
2. Click **Add** (`+`).
3. Set **Interface** to `ether1` (your WAN interface).
4. Click **OK** to apply.
5. Verify that **ether1** receives an IP from your upstream router (check **IP -> DHCP Client**).

**Verification:**

- Under **IP -> DHCP Client**, ensure that **Status** is **bound** for `ether1`.
- Check that an IP address has been assigned to `ether1`.

### 2.5 Enable NAT for Outgoing Traffic

1. Go to **IP -> Firewall -> NAT**.
2. Click **Add** (`+`) to create a new rule.
3. In the **General** tab:
   - Set **Chain** to `srcnat`.
   - Set **Out. Interface** to `ether1` (your WAN interface).
4. In the **Action** tab:
   - Set **Action** to `masquerade`.
5. Click **OK**.

**Verification:**

- Under **IP -> Firewall -> NAT**, confirm that the new rule is added.
- Test internet connectivity from a device connected to the LAN.

---

## 3. Test Internet Connectivity

### 3.1 Test Local Speedtest Server

1. Connect a device to the LAN (e.g., via `ether2`).
2. Ensure the device obtains an IP via DHCP in the `10.0.X.0/24` network.
3. Open a web browser and navigate to `http://192.168.90.10/speedtest/`.
4. Run the speed test and note the results.

### 3.2 Test External Speedtest Server

1. Open a web browser and navigate to `https://speedtest.cesnet.cz/`.
2. Run the speed test and note the results.

---

## 4. Configure Simple Queue (SQ) to Limit Bandwidth

### 4.1 Create a Simple Queue to Limit All Traffic

1. Go to **Queues -> Simple Queues**.
2. Click **Add** (`+`) to create a new queue.
3. In the **General** tab:
   - **Name**: `Limit-All-Traffic`.
   - **Target**: `10.0.X.0/24` (your LAN network).
4. In the **Advanced** tab (if necessary):
   - **Dst. Address**: Leave blank to include all destinations.
5. In the **Max Limit** fields:
   - **Target Upload**: `2M` (limit upload to 2 Mbps).
   - **Target Download**: `4M` (limit download to 4 Mbps).
6. Click **OK**.

**Verification:**

- Under **Queues -> Simple Queues**, confirm that `Limit-All-Traffic` is listed and enabled.
- Run a speed test and observe that download speed is limited to ~4 Mbps and upload to ~2 Mbps.

---

## 5. Test with Multiple Clients and PCQ Queue Type

### 5.1 Adjust Simple Queue with PCQ Queue Type

1. Go to **Queues -> Simple Queues**.
2. Double-click on `Limit-All-Traffic` to edit.
3. In the **Advanced** tab:
   - **Queue Type**:
     - **Target Upload**: Select `pcq-upload-default`.
     - **Target Download**: Select `pcq-download-default`.
4. Click **OK**.

### 5.2 Connect a Second Client

1. Have a neighbor or second device connect to the LAN.
2. Both clients should be active and generating traffic.

**Verification:**

- Ensure both clients have IP addresses in the `10.0.X.0/24` network.

### 5.3 Verify Bandwidth Sharing

1. Run speed tests on both clients simultaneously.
2. Observe how the bandwidth is shared between the clients.

---

## 6. Configure Burst Settings in Simple Queue

### 6.1 Modify Simple Queue to Add Burst Parameters

1. Go to **Queues -> Simple Queues**.
2. Double-click on `Limit-All-Traffic` to edit.
3. In the **Advanced** tab:
   - **Burst Limit**:
     - **Target Upload**: `4M` (burst upload limit).
     - **Target Download**: `6M` (burst download limit).
   - **Burst Threshold**:
     - **Target Upload**: `3M`.
     - **Target Download**: `5M`.
   - **Burst Time**:
     - **Target Upload**: `16s`.
     - **Target Download**: `16s`.
4. Click **OK**.

**Explanation:**

- **Burst Limit**: Maximum allowed bandwidth during a burst.
- **Burst Threshold**: Bandwidth usage level that must be maintained to activate burst.
- **Burst Time**: Duration over which the average bandwidth is calculated.

### 6.2 Verify Burst Functionality

1. Run a speed test and observe the initial higher speeds due to burst.
2. Monitor how the speed reduces to the configured Max Limit after the burst time expires.

---

## 7. Disable Simple Queue (SQ)

1. Go to **Queues -> Simple Queues**.
2. Right-click on `Limit-All-Traffic` and select **Disable**.
3. Alternatively, uncheck the **Enabled** box in the queue settings.

**Verification:**

- Confirm that the `Limit-All-Traffic` queue is disabled.
- Run a speed test to confirm that bandwidth is no longer limited.

---

## 8. Configure Queue Tree (QT) for Advanced Bandwidth Limiting

### 8.1 Mark Traffic in Firewall Mangle

#### 8.1.1 Mark Connections for WEB Traffic

1. Go to **IP -> Firewall -> Mangle**.
2. Click **Add** (`+`) to create a new rule.
3. In the **General** tab:
   - **Chain**: `forward`.
   - **Protocol**: `tcp`.
   - **Dst. Port**: `80,443`.
4. In the **Action** tab:
   - **Action**: `mark connection`.
   - **New Connection Mark**: `web-conn`.
   - **Passthrough**: `yes`.
5. Click **OK**.

#### 8.1.2 Mark Packets for HTTPS Traffic

1. Click **Add** (`+`) to create a new rule.
2. In the **General** tab:
   - **Chain**: `forward`.
   - **Protocol**: `tcp`.
   - **Dst. Port**: `443`.
   - **Connection Mark**: `web-conn`.
3. In the **Action** tab:
   - **Action**: `mark packet`.
   - **New Packet Mark**: `https-pkt`.
   - **Passthrough**: `yes`.
4. Click **OK**.

#### 8.1.3 Mark Packets for HTTP Traffic

1. Click **Add** (`+`) to create a new rule.
2. In the **General** tab:
   - **Chain**: `forward`.
   - **Protocol**: `tcp`.
   - **Dst. Port**: `80`.
   - **Connection Mark**: `web-conn`.
3. In the **Action** tab:
   - **Action**: `mark packet`.
   - **New Packet Mark**: `http-pkt`.
   - **Passthrough**: `yes`.
4. Click **OK**.

**Verification:**

- Under **IP -> Firewall -> Mangle**, confirm that the three rules are added.
- Generate web traffic and check the **Counters** in the Mangle rules to see if packets are being marked.

### 8.2 Create Queue Tree Rules

#### 8.2.1 Create Parent Queue for All WEB Traffic

1. Go to **Queues -> Queue Tree**.
2. Click **Add** (`+`) to create a new queue.
3. Configure:
   - **Name**: `WEB-Traffic`.
   - **Parent**: `global` (or choose the interface if desired).
   - **Packet Mark**: Leave blank.
   - **Max Limit**: `20M` (set to 20 Mbps).
4. Click **OK**.

#### 8.2.2 Create Child Queue for HTTPS Traffic

1. Click **Add** (`+`) to create a new queue.
2. Configure:
   - **Name**: `HTTPS-Traffic`.
   - **Parent**: `WEB-Traffic`.
   - **Packet Mark**: `https-pkt`.
   - **Max Limit**: `15M`.
   - **Limit At**: `7M` (guaranteed minimum bandwidth).
   - **Priority**: `1` (highest priority).
3. Click **OK**.

#### 8.2.3 Create Child Queue for HTTP Traffic

1. Click **Add** (`+`) to create a new queue.
2. Configure:
   - **Name**: `HTTP-Traffic`.
   - **Parent**: `WEB-Traffic`.
   - **Packet Mark**: `http-pkt`.
   - **Max Limit**: `10M`.
   - **Limit At**: `5M`.
   - **Priority**: `2`.
3. Click **OK**.

**Verification:**

- Under **Queues -> Queue Tree**, confirm that the queues are correctly set up.
- Generate web traffic and observe bandwidth allocation in **Queues**.

### 8.3 Verify Bandwidth Limitation

1. Run speed tests for both HTTPS and HTTP traffic.
2. Observe that HTTPS traffic can use up to 15 Mbps and HTTP up to 10 Mbps.
3. Check that HTTPS has higher priority over HTTP when both are active.

---

## 9. Disable Queue Tree (QT)

1. Go to **Queues -> Queue Tree**.
2. Right-click on each queue (`WEB-Traffic`, `HTTPS-Traffic`, `HTTP-Traffic`) and select **Disable**.

**Verification:**

- Confirm that the queues are disabled.
- Run speed tests to ensure traffic is no longer limited by the Queue Tree.

---

## 10. Configure and Test Email Sending

### 10.1 Set up Email Settings

1. Go to **Tools -> Email**.
2. Configure:
   - **Server**: Your SMTP server address (e.g., `smtp.gmail.com`).
   - **Port**: `587` for TLS or `465` for SSL.
   - **Username**: Your email address.
   - **Password**: Your email password or app-specific password.
   - **From**: Your email address.
   - **TLS**: `yes` or `starttls` (depending on your server's requirements).
3. Click **Apply**.

**Note:** For Gmail, you may need to generate an app-specific password and enable access for less secure apps.

### 10.2 Send a Test Email

1. Open the **Terminal**.
2. Execute the command: `/tool e-mail send to="your-email@example.com" subject="RouterOS Test Email" body="This is a test email from RouterOS."`
3. 
3. Check your email inbox for the test message.

**Verification:**

- Confirm receipt of the test email.
- If not received, check the **Logs** for any email-related error messages.

---

## 11. Test Netwatch Tool for Monitoring Neighbors

### 11.1 Set up Netwatch to Monitor a Neighbor

1. Go to **Tools -> Netwatch**.
2. Click **Add** (`+`) to create a new Netwatch entry.
3. Configure:
- **Host**: IP address of the neighbor device to monitor.
- **Interval**: `00:00:10` (checks every 10 seconds).
- **Timeout**: `1000ms` (waits 1 second for a reply).
4. In the **Up Script** field, enter: `log info message="Neighbor is up"`
5. In the **Down Script** field, enter: `log info message="Neighbor is down"`

6. Click **OK**.

### 11.2 Test Netwatch Scripts

1. Disconnect the neighbor device to simulate it going down.
2. Check the **Logs** (under **Log**) to see if the messages appear.
3. Reconnect the device and check for the "Neighbor is up" message.

**Verification:**

- Confirm that the log entries are created when the neighbor's status changes.

---

## 12. Run Profiler and Analyze Processes

### 12.1 Run Profiler

1. Go to **Tools -> Profile**.
2. Observe the processes and their CPU usage.

### 12.2 Understand Process Meanings

- **networking**: Handles network traffic processing.
- **management**: Handles management tasks like WinBox sessions.
- **firewall**: Processes firewall rules.
- **queuing**: Handles traffic queuing and QoS.
- **system**: General system tasks.
- **idle**: CPU idle time.

**Verification:**

- Note which processes consume the most CPU.
- Observe the impact of enabling/disabling services on CPU usage.

---

## 13. Explore Logging Options

### 13.1 Create Logging Rule for Email and Debug Topics

1. Go to **System -> Logging**.
2. Click on the **Rules** tab.
3. Click **Add** (`+`) to create a new logging rule.
4. Configure:
- **Topics**: `email,debug`
- **Action**: `memory` (logs will be stored in memory).
5. Click **OK**.

### 13.2 Send an Email and Check Logs

1. Send a test email as in **Section 10.2**.
2. Go to **Log** and observe entries related to email and debug topics.

**Verification:**

- Confirm that email sending events are logged.
- Check for any debug messages.

---

## 14. Export and Backup Configuration

### 14.1 Export Configuration to a File

1. Open the **Terminal**.
2. Execute the command: `/export file=exercise9-config`

3. The configuration will be saved to a file named `exercise9-config.rsc`.

**Note:** This file contains all configuration commands in text format.

### 14.2 Create a Backup File

1. Go to **Files**.
2. Click **Backup**.
3. Name the backup file (e.g., `exercise9-backup`).
4. Click **Backup** to create the backup file.

**Note:** The backup file is binary and includes sensitive data like passwords.

### 14.3 Download Files for Submission

1. Go to **Files**.
2. Select `exercise9-config.rsc` and `exercise9-backup.backup`.
3. Right-click and select **Download** or drag and drop the files to your desktop.
4. Submit the files to E-Learning as required.

**Verification:**

- Ensure that both files are successfully downloaded.

---

## 15. Reset Router Configuration

1. Go to **System -> Reset Configuration**.
2. Check the box **No Default Configuration**.
3. Click **Reset Configuration** to confirm. The router will reboot with a clean configuration.

---

**Note:** Always handle configurations and backups carefully to avoid exposing sensitive information.






