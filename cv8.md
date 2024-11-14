# RouterOS Configuration Tutorial for Exercise 9

## 1. Reset Router Configuration

1. Open **WinBox** or access the **RouterOS** device.
2. Navigate to **System -> Reset Configuration**.
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

1. Navigate to **IP -> Addresses**.
2. Click **Add** (`+`).
3. In the **Address** field, enter `10.0.X.1/24` (replace `X` with your assigned number).
4. Set the **Interface** to `bridge-lan`.
5. Click **OK**.

**Verification:**

- Under **IP -> Addresses**, confirm that `10.0.X.1/24` is assigned to `bridge-lan`.

### 2.3 Set up DHCP Server on the Bridge

1. Go to **IP -> DHCP Server -> DHCP Setup**.
2. Select `bridge-lan` as the interface.
3. Configure:
   - **DHCP Address Space**: `10.0.X.0/24`.
   - **Gateway for DHCP Network**: `10.0.X.1`.
   - **Address to Give Out**: `10.0.X.2 - 10.0.X.254`.
4. Click **Next** until the setup is complete.

**Verification:**

- Under **IP -> DHCP Server**, confirm that a DHCP server is running on `bridge-lan`.
- Under **IP -> Pool**, confirm that the pool `dhcp_pool1` is set correctly.

### 2.4 Set up DHCP Client on WAN (eth1)

1. Navigate to **IP -> DHCP Client**.
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

## 3. Configure WireGuard Tunnel to Neighbor

### 3.1 Create WireGuard Interface

1. Navigate to **Interfaces -> WireGuard**.
2. Click **Add** (`+`) to create a new WireGuard interface.
3. Set the **Name** to `wg0`.
4. (Optional) Set the **Listen Port** to `51820` (default port) or another unique port.
5. Click **Apply** and **OK**.
6. The **Private Key** and **Public Key** will be generated automatically.

**Verification:**

- Under **Interfaces -> WireGuard**, confirm that `wg0` is listed with a valid **Public Key**.
- The status icon should indicate if the interface is active.

### 3.2 Assign IP Address to WireGuard Interface

1. Go to **IP -> Addresses**.
2. Click **Add** (`+`).
3. In the **Address** field, enter `172.16.X.1/30` (replace `X` with your assigned number).
4. Set the **Interface** to `wg0`.
5. Click **OK**.

**Verification:**

- Under **IP -> Addresses**, confirm that `172.16.X.1/30` is assigned to `wg0`.

### 3.3 Configure WireGuard Peer (Neighbor)

1. Navigate to **Interfaces -> WireGuard** and select `wg0`.
2. Go to the **Peers** tab.
3. Click **Add** (`+`) to create a new peer.
4. Configure the peer settings:
   - **Public Key**: Enter the neighbor's WireGuard public key.
   - **Allowed Address**: `172.16.X.2/32` (neighbor's WireGuard IP).
   - **Endpoint Address**: Enter the neighbor's public IP or hostname.
   - **Endpoint Port**: `51820` (or the port configured on the neighbor's WireGuard).
   - **Persistent Keepalive**: `25` (seconds).
5. Click **OK**.

**Verification:**

- Under **Interfaces -> WireGuard -> Peers**, confirm that the peer is listed and status indicates a connected state.
- Check **Interfaces -> WireGuard** to see if the **RX** and **TX** counters are increasing, indicating traffic flow.

### 3.4 Set Up Firewall Rules for WireGuard

1. Go to **IP -> Firewall -> Filter Rules**.
2. Click **Add** (`+`) to create a new rule.
3. Configure the rule to allow WireGuard traffic:
   - **Chain**: `input`.
   - **Protocol**: `udp`.
   - **Dst. Port**: `51820` (or your WireGuard port).
   - **Action**: `accept`.
4. Click **OK**.

**Verification:**

- Ensure that the firewall rule is active and positioned correctly in the rule list.
- Test connectivity through the WireGuard tunnel.

---

## 4. Configure Routing for WireGuard Tunnel

### 4.1 Add Static Route to Neighbor's LAN

1. Navigate to **IP -> Routes**.
2. Click **Add** (`+`) to create a new route.
3. Configure the route:
   - **Dst. Address**: `192.168.Y.0/24` (neighbor's LAN network).
   - **Gateway**: `172.16.X.2` (neighbor's WireGuard IP).
   - **Distance**: `1`.
4. Click **OK**.

**Verification:**

- Under **IP -> Routes**, confirm that the new route is listed.
- Ensure that the route is active and traffic is being directed through the WireGuard tunnel.

### 4.2 Enable IP Forwarding

1. Go to **IP -> Settings**.
2. Ensure that **IP Forwarding** is enabled.

**Verification:**

- Confirm that IP forwarding is active to allow traffic to pass through the router.

---

## 5. Test Connectivity

### 5.1 Verify Ping to Neighbor's LAN

1. Open the **Terminal**.
2. Execute the command:
   ```
   /ping 192.168.Y.1
   ```
   (Replace `192.168.Y.1` with an IP address of a device in the neighbor's LAN.)

**Expected Result:**

- Successful ping responses indicating connectivity through the WireGuard tunnel.

### 5.2 Test Bandwidth Over WireGuard Tunnel

1. Set up **Bandwidth Test Server** on a device in the neighbor's LAN.
2. On your router, navigate to **Tools -> Bandwidth Test**.
3. Configure the test:
   - **Address**: Neighbor's Bandwidth Test Server IP.
   - **Protocol**: `TCP` or `UDP`.
   - **Direction**: `both` (upload and download).
4. Click **Start** to initiate the test.

**Verification:**

- Observe the bandwidth results.
- Compare with CPU usage using the **Profiler** tool.

---

## 6. Configure and Use Profiler

### 6.1 Run Profiler

1. Navigate to **Tools -> Profile**.
2. Observe the processes and their CPU usage.

**Explanation of Processes:**

- **networking**: Handles network traffic processing.
- **management**: Manages administrative tasks like WinBox sessions.
- **firewall**: Processes firewall rules.
- **queuing**: Manages traffic queuing and QoS.
- **system**: General system tasks.
- **idle**: CPU idle time.

**Verification:**

- Identify which processes are consuming the most CPU.
- Observe the impact of enabling/disabling services on CPU usage.

---

## 7. Export and Backup Configuration

### 7.1 Export Configuration to a File

1. Open the **Terminal**.
2. Execute the command:
   ```
   /export file=exercise9-config
   ```

3. The configuration will be saved to a file named `exercise9-config.rsc`.

**Note:** This file contains all configuration commands in text format.

### 7.2 Create a Backup File

1. Navigate to **Files**.
2. Click **Backup**.
3. Name the backup file (e.g., `exercise9-backup`).
4. Click **Backup** to create the backup file.

**Note:** The backup file is binary and includes sensitive data like passwords.

### 7.3 Download Files for Submission

1. Go to **Files**.
2. Select `exercise9-config.rsc` and `exercise9-backup.backup`.
3. Right-click and select **Download** or drag and drop the files to your desktop.
4. Submit the files to E-Learning as required.

**Verification:**

- Ensure that both files are successfully downloaded.

---

## 8. Reset Router Configuration

1. Navigate to **System -> Reset Configuration**.
2. Check the box **No Default Configuration**.
3. Click **Reset Configuration** to confirm. The router will reboot with a clean configuration.

---

**Note:** Always handle configurations and backups carefully to avoid exposing sensitive information.
