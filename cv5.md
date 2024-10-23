# RouterOS Configuration Tutorial for Exercise 5

## 1. Reset RouterOS Configuration
1. Open **WinBox** or access the **RouterOS** device.
2. Go to **System -> Reset Configuration**.
3. Check the box for **No Default Configuration** to start from a clean state.
4. Click **Reset Configuration** to confirm. The router will reboot.

---

## 2. Basic Configuration (WAN and LAN Setup)

### 2.1 Set up DHCP Client on WAN (ether1)
1. Go to **IP -> DHCP Client**.
2. Click **Add** (`+`).
3. Set **Interface** to `ether1` (your WAN interface).
4. Click **OK** to apply.
5. Verify that **ether1** receives an IP from your upstream router (check **IP -> DHCP Client**).

### 2.2 Create a Bridge for LAN (ether2, ether3, ether4)
1. Go to **Interfaces -> Bridge**.
2. Click **Add** (`+`) and name it `bridge-lan`.
3. Go to the **Ports** tab.
4. Add **ether2**, **ether3**, and **ether4** to the `bridge-lan`:
   - For each, click **Add** (`+`), set **Interface** to the respective port, and set **Bridge** to `bridge-lan`.
   - Click **OK** after adding each interface.

### 2.3 Assign IP Address to the Bridge
1. Go to **IP -> Addresses**.
2. Click **Add** (`+`).
3. In the **Address** field, enter `10.0.X.1/24` (where `X` is your assigned number).
4. Set the **Interface** to `bridge-lan`.
5. Click **OK**.

### 2.4 Set up DHCP Server for the LAN
1. Go to **IP -> DHCP Server -> DHCP Setup**.
2. Select **bridge-lan** as the interface.
3. Configure:
   - **DHCP Address Range**: `10.0.X.0/24`.
   - **Gateway**: `10.0.X.1`.
   - **IP Pool**: `10.0.X.2 - 10.0.X.254`.
4. Click **Next** to finish the setup.

### 2.5 Enable NAT for Internet Access
1. Go to **IP -> Firewall -> NAT**.
2. Click **Add** (`+`) to create a new rule.
3. In the **General** tab:
   - Set **Chain** to `srcnat`.
   - Set **Out. Interface** to `ether1` (your WAN interface).
4. In the **Action** tab:
   - Set **Action** to `masquerade`.
5. Click **OK**.

---

## 3. Wireless Access Point Setup

### 3.1 Enable Wireless Interface
1. Go to **Interfaces -> Wireless**.
2. Select the **wireless interface** (e.g., `wlan1`) and enable it.
   - If disabled, right-click on `wlan1` and select **Enable**.

### 3.2 Configure the Wireless Access Point (AP)
1. Go to **Interfaces -> Wireless**.
2. Click on the **wireless interface** (`wlan1`).
3. In the **Wireless** tab:
   - Set **Mode** to `AP Bridge` (to act as an Access Point).
   - Set the **SSID** to `MOS-2024-X` (the name of the Wi-Fi network).
   - Set any **security profile** (WPA2, etc.) to secure the network.

### 3.3 Assign IP Address to the Wireless Interface (wlan1)
1. Go to **IP -> Addresses**.
2. Click **Add** (`+`).
3. In the **Address** field, enter `10.1.X.1/24` (replace `X` with your number).
4. Set the **Interface** to `wlan1`.
5. Click **OK**.

### 3.4 Set Up DHCP Server for the Wireless Network (wlan1)
1. Go to **IP -> DHCP Server -> DHCP Setup**.
2. Select **wlan1** as the interface.
3. Configure:
   - **DHCP Address Range**: `10.1.X.0/24`.
   - **Gateway**: `10.1.X.1` (the IP assigned to `wlan1`).
   - **IP Pool**: `10.1.X.2 - 10.1.X.254`.
4. Click **Next** to complete the setup.

---

## 4. Test Internet Access
1. Connect a **wireless device** (e.g., phone, laptop) to the **SSID** (`MOS-2024-X`).
2. Ensure the device is set to **obtain an IP address via DHCP**.
3. **Ping** `8.8.8.8` from the wireless device or wired devices connected to the LAN.
4. Open a web browser and verify that the device can browse the internet.

---

## 5. Verify Firewall Connections
1. Go to **IP -> Firewall -> Connections**.
2. Review active connections and check how packets are being processed.

---

## 6. Disable Connection Tracking (Verification)
1. Go to **IP -> Firewall -> Connections**.
2. Click on the **Tracking** tab and disable it by setting **Tracking** to `No`.
3. Test web access or **ping** to `8.8.8.8`.
4. **Result**: Connection tracking being off will likely break NAT and other functionalities like **firewall rules**.
5. Re-enable **Tracking** after testing.

---

## 7. Safe Mode for Firewall Configuration
1. It is highly recommended to use **Safe Mode** while configuring the firewall.
   - In **WinBox**, click the **Safe Mode** button at the top to ensure you don’t accidentally lock yourself out.

---

## 8. Firewall Configuration (Securing Router)

### 8.1 Allow Established and Related Connections (Chain: INPUT)
1. Go to **IP -> Firewall -> Filter Rules**.
2. Click **Add** (`+`).
3. In the **General** tab:
   - Set **Chain** to `INPUT`.
   - Set **Connection State** to `established, related`.
4. In the **Action** tab:
   - Set **Action** to `accept`.
5. Click **OK**.

---

### 8.2 Drop Traffic from "Attack" Address List
1. **Create the "Attack" Address List**:
   - Go to **IP -> Firewall -> Address Lists**.
   - Click **Add** (`+`).
   - Set **Name** to `Attack`.
   - Leave the **Address** field empty for now (we will add offending IPs dynamically later).
   - Click **OK**.
   
2. **Create the Drop Rule**:
   - Go to **IP -> Firewall -> Filter Rules**.
   - Click **Add** (`+`).
   - In the **General** tab:
     - Set **Chain** to `INPUT`.
     - Set **Src. Address List** to `Attack`.
   - In the **Action** tab:
     - Set **Action** to `drop`.
   - Click **OK**.

---

### 8.3 Create and Accept Traffic from "Support" Address List
1. **Create the "Support" Address List**:
   - Go to **IP -> Firewall -> Address Lists**.
   - Click **Add** (`+`).
   - Set **Name** to `Support`.
   - In the **Address** field, add:
     - `192.168.254.254`
   - Repeat for:
     - `192.168.254.100`
     - `10.0.X.0/24` (replace `X` with your assigned LAN number).
   - Click **OK**.
   
2. **Allow Traffic from "Support" List**:
   - Go to **IP -> Firewall -> Filter Rules**.
   - Click **Add** (`+`).
   - In the **General** tab:
     - Set **Chain** to `INPUT`.
     - Set **Src. Address List** to `Support`.
   - In the **Action** tab:
     - Set **Action** to `accept`.
   - Click **OK**.

---

### 8.4 ICMP Rules (Chain: INPUT)
#### ICMP Echo Reply
1. Go to **IP -> Firewall -> Filter Rules**.
2. Click **Add** (`+`).
3. In the **General** tab:
   - Set **Chain** to `INPUT`.
   - Set **Protocol** to `icmp`.
4. In the **Action** tab:
   - Set **ICMP Type** to `echo-reply`.
   - Set **Action** to `accept`.
6. Click **OK**.

#### Rate-Limit ICMP Echo Requests
1. Go to **IP -> Firewall -> Filter Rules**.
2. Click **Add** (`+`).
3. In the **General** tab:
   - Set **Chain** to `INPUT`.
   - Set **Protocol** to `icmp`.
4. In the **Advanced** tab:
   - Set **ICMP Type** to `echo-request`.
5. In the **Extra** tab:
   - Set **Limit** to `1 packet per second, burst 5`.
6. In the **Action** tab:
   - Set **Action** to `accept`.
7. Click **OK**.

---

### 8.5 Port Scan and SYN Flood Detection
1. **Create SYN Flood Detection**:
   - Go to **IP -> Firewall -> Filter Rules**.
   - Click **Add** (`+`).
   - In the **General** tab:
     - Set **Chain** to `INPUT`.
     - Set **Protocol** to `tcp`.
     - Set **TCP Flags** to `syn`.
   - In the **Advanced** tab:
     - Set **Src. Limit** to `20,5` (limit source to 20 connections per 5 seconds).
     - Set **Src. Address List** to `Attack`.
   - In the **Action** tab:
     - Set **Action** to `add-src-to-address-list`.
     - Set **Address List** to `Attack`.
   - Click **OK**.

---

### 8.6 Catch-All Rule to Log and Drop Remaining Traffic
1. Go to **IP -> Firewall -> Filter Rules**.
2. Click **Add** (`+`).
3. In the **General** tab:
   - Set **Chain** to `INPUT`.
4. In the **Action** tab:
   - Set **Action** to `drop`.
   - Enable **Log**.
5. Click **OK**.

---

## 9. Forward Chain Configuration (Securing Network)
1. **Allow Established and Related Connections**:
   - Set up a rule similar to **8.1**, but set **Chain** to `FORWARD`.

2. **Allow Wi-Fi Clients (10.1.X.0/24) to Access Only the Internet**:
   1. Go to **IP -> Firewall -> Filter Rules**.
   2. Click **Add** (`+`).
   3. Set **Chain** to `FORWARD`.
   4. In the **General** tab:
      - Set **Src. Address** to `10.1.X.0/24`.
      - Set **Dst. Address** to `!10.0.X.0/24` (deny access to LAN).
   5. In the **Action** tab:
      - Set **Action** to `accept`.
   6. Click **OK**.

3. **Allow LAN Clients (10.0.X.0/24) to Access Everything**:
   1. Go to **IP -> Firewall -> Filter Rules**.
   2. Click **Add** (`+`).
   3. Set **Chain** to `FORWARD`.
   4. In the **General** tab:
      - Set **Src. Address** to `10.0.X.0/24`.
   5. In the **Action** tab:
      - Set **Action** to `accept`.
   6. Click **OK**.

---

## 10. Additional NAT Rules

### 10.1 Masquerade for Internet Access (Already Done)

### 10.2 DST-NAT for Remote Desktop (RDP)
1. Go to **IP -> Firewall -> NAT**.
2. Click **Add** (`+`).
3. In the **General** tab:
   - Set **Chain** to `dstnat`.
   - Set **Dst. Port** to `33389`. (Protocol to tcp)
4. In the **Action** tab:
   - Set **Action** to `dst-nat`.
   - Set the **To Addresses** field to your PC’s IP (e.g., `10.0.X.2`).
   - Set the **To Ports** field to `3389` (RDP).
5. Click **OK**.

### 10.3 DST-NAT for HTTP Traffic from Wi-Fi
1. Go to **IP -> Firewall -> NAT**.
2. Click **Add** (`+`).
3. In the **General** tab:
   - Set **Chain** to `dstnat`.
   - Set **Protocol** to `tcp`.
   - Set **Dst. Port** to `80`.
   - Set **Src. Address** to `10.1.X.0/24`.
4. In the **Action** tab:
   - Set **Action** to `dst-nat`.
   - Set **To Address** to `router’s IP`.
   - Set **To Port** to `80`.
5. Click **OK**.

---

## 11. Backup Configuration
1. Go to **System -> Backup** and create a backup of your configuration.
2. Save the configuration file and upload it to your e-learning platform if required.

---

### 1. Port Knocking
#### Step 1: Set Up Port Knocking Phases
1. Go to **IP -> Firewall -> Filter Rules**.
2. Click **Add** (`+`) to create a new rule for **Phase 1**:
   - In the **General** tab:
     - Set **Chain** to `input`.
     - Set **Protocol** to `tcp`.
     - Set **Dst. Port** to `5000`.
   - In the **Action** tab:
     - Set **Action** to `add-src-to-address-list`.
     - Set **Address List** to `PortKnock1`.
     - Set **Timeout** to `20s`.
   - Click **OK**.

3. Repeat to create **Phase 2**:
   - In the **General** tab:
     - Set **Chain** to `input`.
     - Set **Protocol** to `tcp`.
     - Set **Dst. Port** to `9999`.
     - In the **Advanced** tab, set **Src. Address List** to `PortKnock1`.
   - In the **Action** tab:
     - Set **Action** to `add-src-to-address-list`.
     - Set **Address List** to `PortKnock2`.
     - Set **Timeout** to `20s`.
   - Click **OK**.

4. Repeat to create **Phase 3**:
   - In the **General** tab:
     - Set **Chain** to `input`.
     - Set **Protocol** to `tcp`.
     - Set **Dst. Port** to `195`.
     - In the **Advanced** tab, set **Src. Address List** to `PortKnock2`.
   - In the **Action** tab:
     - Set **Action** to `add-src-to-address-list`.
     - Set **Address List** to `Support`.
   - Click **OK**.

#### Step 2: Test Port Knocking
1. Try accessing the router’s management interface after performing the correct knock sequence.
2. Open a web browser and navigate to `http://[ROUTER_IP]:port`.

---

### 2. SSH Brute Force Protection
#### Step 1: Create Rules to Detect Brute Force
1. Go to **IP -> Firewall -> Filter Rules**.
2. Click **Add** (`+`) to create a detection rule:
   - In the **General** tab:
     - Set **Chain** to `input`.
     - Set **Protocol** to `tcp`.
     - Set **Dst. Port** to `22` (SSH).
     - Set **Connection State** to `new`.
   - In the **Action** tab:
     - Set **Action** to `add-src-to-address-list`.
     - Set **Address List** to `SSH-Brute`.
     - Set **Timeout** to `1h` (or any preferred duration).
   - Click **OK**.

#### Step 2: Block Detected Brute Force IPs
1. Go to **IP -> Firewall -> Filter Rules**.
2. Click **Add** (`+`) to create a block rule:
   - In the **General** tab:
     - Set **Chain** to `input`.
     - Set **Src. Address List** to `SSH-Brute`.
   - In the **Action** tab:
     - Set **Action** to `drop`.
   - Click **OK**.

#### Step 3: Test SSH Protection
1. Create a test user (`test/test`) in **System -> Users**.
2. Attempt to connect multiple times with **incorrect credentials** using **PuTTY** to trigger the brute force protection.
3. Verify that your IP is added to the `SSH-Brute` address list and is being blocked.

---

### 3. Block Access to a Specific Website (idnes.cz)
#### Step 1: Create Layer 7 Protocol for Domain Matching
1. Go to **IP -> Firewall -> Layer7 Protocols**.
2. Click **Add** (`+`) to create a new rule:
   - Set **Name** to `BlockIdnes`.
   - In the **Regexp** field, enter:
     ```
     ^.*idnes\.cz.*$
     ```
   - Click **OK**.

#### Step 2: Create Firewall Rule to Block idnes.cz
1. Go to **IP -> Firewall -> Filter Rules**.
2. Click **Add** (`+`) to create a block rule:
   - In the **General** tab:
     - Set **Chain** to `forward`.
   - In the **Advanced** tab:
     - Set **Layer7 Protocol** to `BlockIdnes`.
   - In the **Action** tab:
     - Set **Action** to `drop`.
   - Click **OK**.

#### Step 3: Test Website Blocking
1. On a device connected to your **LAN** or **Wi-Fi**, try accessing **idnes.cz**.
2. The site should be blocked.
