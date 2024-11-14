# Modern Network Technologies - Exercise 3

## 1. Log in to the Router

1. Use the default credentials to log in (username: `admin`, password: empty or as per your setup).

---

## 2. Reset Router Configuration

1. Go to **System -> Reset Configuration**.
2. Check the box **No Default Configuration** to start from a clean state.
3. Click **Reset Configuration** to confirm. The router will reboot.

---

## 3. Check Current Port Settings

1. Check the current configuration of ports to identify if they are set as switch, bridge, or independent ports.

---

## 4. Create Bridge for WAN

1. Go to **Interfaces -> Bridge**.
2. Click **Add** (`+`) and name it `Bridge-WAN`.
3. Go to **IP -> DHCP Client** and assign it to `Bridge-WAN`.

### Add Ports to Bridge-WAN

1. Go to **Bridge -> Ports** and add **eth1** and **eth2** to `Bridge-WAN`.
2. Connect your PC to **eth2** and observe the IP it receives from the DHCP server.

**Question:** What happened? Why? Check the IP your computer received from the DHCP server.

---

## 5. Create Bridge for LAN

1. Go to **Interfaces -> Bridge**.
2. Click **Add** (`+`) and name it `Bridge-LAN`.

### Add Ports to Bridge-LAN

1. Go to **Bridge -> Ports** and add **eth3**, **eth4**, and any other necessary ports to `Bridge-LAN`.
2. Connect your PC to one of these ports.

---

## 6. Set Up DHCP Server on Bridge-LAN

1. Go to **IP -> DHCP Server -> DHCP Setup**.
2. Select `Bridge-LAN` as the interface.
3. Configure the DHCP server to assign IPs from the range `10.0.X.0/24`.

**Verification:**

- Confirm that your PC has received the correct IP address from the DHCP server.

---

## 7. Disable DHCP Client if Not First Router in Network

1. If you have an active DHCP client on **eth1** or `Bridge-WAN` and are not the first router with internet access, disable it.

---

## 8. Configure Static Routing (Figure 2.1)

1. Set up IP addresses on each router’s **eth1** interface as shown in Figure 2.1.
2. Configure static routes for inter-router communication using eth1 on the same switch (no additional cables needed).

**Verification:**

- Use **ping** to test connectivity between PCs on different LANs.
- Use **traceroute** to check the route to the internet (e.g., `8.8.8.8`) to ensure correct path.

---

## 9. Disable Static Routes and Re-enable DHCP Client

1. Disable any static routing rules.
2. Re-enable the DHCP client on **eth1**.

---

## 10. Update Network Topology (Figure 2.2)

Ensure that the network is configured according to the logical layout in Figure 2.2.

---

## 11. Enable RIP Protocol for Dynamic Routing

### Enable RIP on Routers

1. Go to **Routing -> RIP** on each router.
2. Enable RIP to exchange routing information for connected networks.
3. Set up the `192.168.10.0/24` network for RIP.

**Verification:**

- Test with **ping** and **traceroute** to verify connectivity between LANs and the internet.
- Ensure NAT is configured as per previous exercises to allow internet access.

---

## 12. Export and Backup Configuration

### Export Configuration to a File

1. Open the **Terminal**.
2. Execute the command:
   ```
   /export file=exercise3-config
   ```

3. The configuration will be saved to a file named `exercise3-config.rsc`.

### Create a Backup File

1. Go to **Files**.
2. Click **Backup**.
3. Name the backup file (e.g., `exercise3-backup`).
4. Click **Backup** to create the backup file.

**Note:** The backup file is binary and includes sensitive data like passwords.

### Download Files for Submission

1. Go to **Files**.
2. Select `exercise3-config.rsc` and `exercise3-backup.backup`.
3. Right-click and select **Download** or drag and drop the files to your desktop.
4. Submit the files to E-Learning as required.

---

## 13. Reset Router Configuration

1. Go to **System -> Reset Configuration**.
2. Uncheck the box **No Default Configuration** to restore the router’s original setup.
3. Click **Reset Configuration** to confirm. The router will reboot with default settings.

---

**Note:** Carefully handle configurations and backups to avoid exposing sensitive information.
