# RouterOS Configuration Tutorial for Exercise 6

## 1. Reset RouterOS Configuration
1. Open **WinBox** or access the **RouterOS** device.
2. Go to **System -> Reset Configuration**.
3. Check **No Default Configuration** to start from a clean state.
4. Click **Reset Configuration**. The router will reboot.

---

## 2. LAN Bridge Configuration

### 2.1 Configure LAN Bridge (ether2, ether3, ether4)
1. Go to **Interfaces -> Bridge**.
2. Click **Add** (`+`) and name it `bridge-lan`.
3. Go to the **Ports** tab.
4. Add **ether2**, **ether3**, and **ether4** to the bridge:
   - For each, click **Add** (`+`), set **Interface** to the respective port, and set **Bridge** to `bridge-lan`.
   - Click **OK** after each.

### 2.2 Set Up DHCP Server on LAN
1. Go to **IP -> DHCP Server -> DHCP Setup**.
2. Select **bridge-lan**.
3. Configure:
   - **Network**: `10.0.X.0/24` (replace `X` with your assigned number)
   - **Gateway**: `10.0.X.1`
   - **Address Pool**: `10.0.X.2 - 10.0.X.254`
4. Complete the setup by following the prompts.

---

## 3. Wireless Client Configuration

### 3.1 Connect WiFi Client to Lab Network
1. Go to **Interfaces -> Wireless**.
2. Select **wlan1** and configure:
   - **SSID**: `Lab537`
   - **Password**: `heslo2023`
3. Go to **IP -> DHCP Client** and add a new client on **wlan1**:
   - Set **Use Peer DNS** to `yes`.
   - **Default Route** should be unchecked.

### 3.2 Static Default Route via WiFi Interface
1. Go to **IP -> Routes**.
2. Click **Add** (`+`).
3. Configure the route:
   - **Dst. Address**: `0.0.0.0/0`
   - **Gateway**: Set the gateway IP obtained from the DHCP client on **wlan1**.[how to](https://github.com/martinszuc/fekt-mos/blob/main/cv6.md#100-gateway-ip-from-dhcp-client-on-wlan1)
   - **Distance**: `10`
4. Click **OK** to save.

---

## 4. Router Identity Configuration
1. Go to **System -> Identity**.
2. Set **Name** to `Router-X` (replace `X` with your assigned workstation number).
3. Click **OK** to apply.

---

## 5. PPP Configuration

### 5.1 IP Pool for PPP Clients
1. Go to **IP -> Pool**.
2. Click **Add** (`+`).
3. Set **Name** to `ppp-pool`.
4. Set **Addresses** to `10.111.X.0/24` (replace `X` with your workstation number).
5. Click **OK**.

### 5.2 Create PPP Profiles
1. Go to **PPP -> Profiles**.
2. Duplicate the **default-encryption** profile twice to create:
   - A **server** profile.
   - A **client** profile.
3. Assign IP addresses to the **server** profile only.
4. Click **OK** to save each profile.

### 5.3 Create PPP User
1. Go to **PPP -> Secrets**.
2. Click **Add** (`+`) and configure:
   - **Name**: `jmenoX` (replace `X` with your workstation number)
   - **Password**: `hesloX`
   - **Service**: `ppp`
   - **Profile**: Choose the server or client profile as appropriate.
3. Click **OK** to save.

---

## 6. PPPoE Client Configuration

### 6.1 Configure PPPoE Client on ether1
1. Go to **Interfaces -> PPPoE Client**.
2. Click **Add** (`+`).
3. Set **Name** to `PPPoE-MMOS`.
4. Configure **Dial Out** settings:
   - **User**: `jmeno0`
   - **Password**: `heslo0`
   - **Service Name**: `PPPoE-MMOS`
5. Click **OK**.

### 6.2 Configure Static Default Route via PPPoE
1. Go to **IP -> Routes**.
2. Click **Add** (`+`).
3. Configure the route:
   - **Dst. Address**: `0.0.0.0/0`
   - **Gateway**: Set the gateway IP obtained from the PPPoE client.
   - **Distance**: `20`
4. Click **OK** to save.

### 6.3 NAT for PPPoE Client
1. Go to **IP -> Firewall -> NAT**.
2. Click **Add** (`+`) and configure:
   - **Chain**: `srcnat`
   - **Out. Interface**: `PPPoE-MMOS`
   - **Action**: `masquerade`
3. Click **OK**.

---

## 7. Testing and Verification

### 7.1 Test Internet Access
1. Open a terminal and perform a traceroute to `8.8.8.8` to verify connectivity.

### 7.2 Disable WiFi and Retest
1. Go to **Interfaces -> Wireless** and disable **wlan1**.
2. Perform a traceroute to `8.8.8.8` again to confirm that PPPoE routing is functioning.

3. Re-enable **wlan1** after the test.

---

## 8. PPTP Server Setup

### 8.1 Enable PPTP Server
1. Go to **PPP -> PPTP Server**.
2. Enable the server and assign the appropriate profile.
3. Click **OK** to save.

### 8.2 Configure PPTP Client
1. Go to **Interfaces -> PPTP Client** and add a new client.
2. Configure:
   - **Server Address**: Set to the neighbor's router IP.
   - **User**: Set to the neighbor’s PPP credentials.
   - **Profile**: Select the appropriate client profile.
3. Click **OK** to establish the connection.

### 8.3 Configure Local Network Routing to Neighbor
1. Go to **IP -> Routes**.
2. Add a new route for the neighbor’s local network.

---

## 9. Final Testing

### 9.1 Verify Connectivity with Neighbor
1. Use **ping** to test connectivity between devices on both networks.

### 9.2 Export and Backup Configuration
1. Go to **Files**.
2. Click **Backup** and save your configuration.

---

## 10. Reset Router
1. Reset the router using **System -> Reset Configuration** without **No Default Configuration**.



addditional info:
### 100. Gateway IP from DHCP Client on wlan1

1. **Open the DHCP Client Status**:
   - Go to **IP -> DHCP Client**.
   - Locate the DHCP client entry for `wlan1`.
   - Double-click on this entry to view its status.

2. **View the Gateway IP**:
   - In the DHCP Client Status window, check the **Gateway** field.
   - Note down the IP address displayed here; this is the gateway IP provided by the DHCP server.

3. **Set the Gateway IP in Static Routes**:
   - Go to **IP -> Routes**.
   - Click **Add** (`+`) to create a new static route.
   - Set the **Dst. Address** to `0.0.0.0/0`.
   - In the **Gateway** field, enter the gateway IP you noted down in Step 2.
   - Set **Distance** to `10` (or your required metric).
   - Click **OK** to save the route.

Now, the router will use this gateway IP from the `wlan1` interface for routing.
