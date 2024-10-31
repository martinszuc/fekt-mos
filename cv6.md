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
   - **Gateway**: Set the gateway IP obtained from the PPPoE client. [how to](https://github.com/martinszuc/fekt-mos/blob/main/cv6.md#101-gateway-ip-from-pppoe-client)
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

## 8. PPTP Server and Client Setup

In this section, we’ll configure a PPTP server and client to establish a secure connection between neighboring routers. The purpose is to enable communication and routing between your local network and your neighbor’s network, creating a simple VPN-like setup for testing and collaboration.

### 8.1 Enable PPTP Server
1. Go to **PPP -> PPTP Server** in **WinBox**.
2. Enable the **PPTP Server** by checking the box labeled **Enabled**.
3. In the **Profile** dropdown, select the appropriate **server profile** (this should include IP configurations specific to the server side).
4. Click **OK** to save and activate the server.

> **Note**: Enabling the PPTP server allows your router to accept connections from a PPTP client configured on your neighbor’s router.

### 8.2 Configure PPTP Client (Connection to Neighbor’s PPTP Server)
1. **Coordinate with Your Neighbor**:
   - Decide on IP addresses and credentials for each router.
   - Ensure both routers have **active PPTP servers and clients** to allow two-way communication.
   - Identify the IP address of the neighbor’s PPTP server interface for configuration.

2. **Add PPTP Client**:
   - Go to **Interfaces -> PPTP Client** and click **Add** (`+`).
   - Configure the client connection:
     - **Server Address**: Enter your neighbor’s router IP (PPTP server address).
     - **User**: Enter the username provided by your neighbor.
     - **Password**: Enter the password provided by your neighbor.
     - **Profile**: Select the client profile created for PPTP connections.
   - Click **OK** to establish the connection.

> **Result**: Your router will connect to the neighbor’s PPTP server, creating a secure, tunnel-like connection between both routers.

### 8.3 Configure Local Network Routing to Neighbor
To enable routing between local networks (LANs) of both routers:

1. Go to **IP -> Routes**.
2. Click **Add** (`+`) to create a new route.
3. Set the following:
   - **Dst. Address**: Enter your neighbor’s LAN network address (e.g., `192.168.Y.0/24`).
   - **Gateway**: Select the PPTP client interface or enter the PPTP connection IP.
4. Click **OK** to save.

> **Objective**: This route directs traffic to the neighbor’s LAN through the PPTP connection, allowing devices on both networks to communicate.

---

## 9. Final Testing and Backup

### 9.1 Verify Connectivity with Neighbor
1. From your router, use **ping** to test connectivity to devices in your neighbor’s LAN.
2. Confirm that you can communicate with your neighbor’s network successfully.

> **Goal**: Pinging confirms that the connection and routing are set up correctly, allowing devices to exchange data over the PPTP link.

### 9.2 Export and Backup Configuration
1. Go to **Files** in **WinBox**.
2. Click **Backup** and save the configuration file.
3. **Submit the backup** to your e-learning platform if required.

> **Reason**: Backing up the configuration ensures you have a record of your setup for future reference or restoration.

---

## 10. Reset Router

After completing the exercise, **reset the router** to its default settings:

1. Go to **System -> Reset Configuration**.
2. Uncheck **No Default Configuration** to restore the router’s original setup.
3. Click **Reset Configuration** and allow the router to reboot.

> **Purpose**: Resetting the router clears the custom configuration, preparing the device for the next user or exercise.




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


### 101. Gateway IP from PPPoE Client

To find and set the gateway IP from the PPPoE client in **WinBox**:

1. **Open the PPPoE Client Status**:
   - Go to **Interfaces -> PPP** in **WinBox**.
   - Select the **PPPoE client** interface you've configured (e.g., `PPPoE-MMOS`).
   - Double-click on the interface to open its properties.

2. **Locate the Gateway IP**:
   - In the **Status** tab, find the **Gateway** field.
   - Note the IP address displayed here; this is the gateway IP assigned by the PPPoE client.

3. **Set the Gateway in Routes**:
   - Go back to **IP -> Routes**.
   - Add a new route or edit an existing one.
   - In the **Gateway** field, enter the IP address obtained from Step 2.
   - Set the **Distance** field to your desired metric (e.g., `20`).

4. Click **OK** to save the route configuration.

This completes setting up a static default route using the gateway IP from the PPPoE client.
