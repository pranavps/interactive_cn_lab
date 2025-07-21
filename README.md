# Practical Assignment: Building a Basic Network and Application Services on VMs

## Objective

This assignment aims to provide hands-on experience with fundamental networking concepts, including VM setup, network configuration (now with DHCP), and the implementation of basic client-server applications using TCP and UDP sockets. You will deploy and interact with common application-layer protocols (HTTP, FTP, DNS), implement a basic firewall with a DMZ and a mobile network segment, and prepare for future packet analysis exercises.

## Tools Required

- **Oracle VirtualBox:** For creating and managing virtual machines
- **Ubuntu Server (24.04.2 LTS):** A lightweight Linux distribution suitable for server roles. Download the ISO image
- **Ubuntu Desktop (24.04.2 LTS):** For a general-purpose client VM and a simulated mobile device. Download the ISO image
- **Kali Linux (latest version):** For a specialized client VM focused on network analysis. Download the ISO image
- **Endian Firewall (EFW) 3.3.2:** A powerful open-source firewall/router distribution. Download the ISO image
- **Python 3:** For implementing custom client-server applications
- **Basic Linux Command Line Knowledge:** Essential for navigating, configuring, and troubleshooting
- **Network Utilities:** `ping`, `ip a` (or `ifconfig`), `netstat -tuln` (or `ss -tuln`), `dig` (or `nslookup`), `traceroute`. Consider `tcpdump` or Wireshark for advanced debugging

## Deliverables

1. **Report (PDF):** A document detailing your steps, configurations, observations, and answers to questions. Include screenshots where specified
2. **Source Code:** All Python scripts developed for the assignment

---

## Part 1: Virtual Machine Setup and Network Infrastructure

In this part, you will set up multiple virtual machines and configure a network topology that includes a dedicated firewall with a DMZ and a simulated mobile network.

### 1.1 VM Creation

Create five new Virtual Machines in Oracle VirtualBox:

#### VM Name: `ServerVM`
- **OS:** Ubuntu Server (24.04.2 LTS, 64-bit)
- **Memory:** At least 2GB (2048 MB)
- **Hard Disk:** At least 20GB dynamically allocated
- **Network Adapter 1 (NAT):** Keep the default NAT adapter for initial internet access (for installing packages). This will be removed later
- **Network Adapter 2 (Internal Network):** Add a second adapter. Configure it as an **Internal Network** named `LAN_NET`. This will connect to the firewall's GREEN (LAN) interface

#### VM Name: `ClientVM`
- **OS:** Ubuntu Desktop (24.04.2 LTS, 64-bit)
- **Memory:** At least 2GB (2048 MB)
- **Hard Disk:** At least 20GB dynamically allocated
- **Network Adapter 1 (NAT):** Keep the default NAT adapter for initial internet access. This will be removed later
- **Network Adapter 2 (Internal Network):** Add a second adapter. Configure it as an **Internal Network** named `LAN_NET` (same as ServerVM)

#### VM Name: `KaliClientVM`
- **OS:** Kali Linux (latest version, 64-bit)
- **Memory:** At least 2GB (2048 MB)
- **Hard Disk:** At least 30GB dynamically allocated
- **Network Adapter 1 (NAT):** Keep the default NAT adapter for initial internet access. This will be removed later
- **Network Adapter 2 (Internal Network):** Add a second adapter. Configure it as an **Internal Network** named `LAN_NET` (same as ServerVM and ClientVM)

#### VM Name: `MobileClientVM`
- **OS:** Ubuntu Desktop (24.04.2 LTS, 64-bit)
- **Memory:** At least 2GB (2048 MB)
- **Hard Disk:** At least 20GB dynamically allocated
- **Network Adapter 1 (NAT):** Keep the default NAT adapter for initial internet access. This will be removed later
- **Network Adapter 2 (Internal Network):** Add a second adapter. Configure it as an **Internal Network** named `BLUE_NET`. This will connect to the firewall's BLUE (Wireless/VPN) interface

#### VM Name: `FirewallVM`
- **OS:** Endian Firewall (EFW) 3.3.2 (64-bit)
- **Memory:** At least 1GB (1024 MB)
- **Hard Disk:** At least 10GB dynamically allocated
- **Network Adapter 1 (Host-Only Adapter):** This will be the **RED (WAN)** interface. Create a VirtualBox Host-Only Network (e.g., `vboxnet0`, typically in the 192.168.56.0/24 range). Note down its IP range
- **Network Adapter 2 (Internal Network):** This will be the **GREEN (LAN)** interface. Configure it as an **Internal Network** named `LAN_NET` (same as ServerVM, ClientVM, KaliClientVM)
- **Network Adapter 3 (Internal Network):** Add a third adapter. Configure it as an **Internal Network** named `DMZ_NET`. This will be the **ORANGE (DMZ)** interface
- **Network Adapter 4 (Internal Network):** Add a fourth adapter. Configure it as an **Internal Network** named `BLUE_NET` (same as MobileClientVM). This will be the **BLUE (Wireless/VPN)** interface

### 1.2 Ubuntu/Kali/EFW Installation

#### Install Ubuntu Server on ServerVM:
- Choose a strong password for your user
- Select "Install OpenSSH server"
- No desktop environment needed

#### Install Ubuntu Desktop on ClientVM and MobileClientVM:
- Choose a strong password for your user
- Select "Install OpenSSH server" (optional)
- A desktop environment will be installed

#### Install Kali Linux on KaliClientVM:
- Follow the standard Kali Linux installation process. Choose a strong password
- A desktop environment and many security/network tools will be installed

#### Install Endian Firewall (EFW) on FirewallVM:
- Boot from the EFW ISO
- Follow the guided installation process. You will be prompted to configure network interfaces during setup
- **Assign Interfaces:**
  - Identify your RED (WAN) interface (connected to Host-Only Adapter)
  - Identify your GREEN (LAN) interface (connected to Internal Network LAN_NET)
  - Identify your ORANGE (DMZ) interface (connected to Internal Network DMZ_NET)
  - Identify your BLUE (Wireless/VPN) interface (connected to Internal Network BLUE_NET)
- **Configure RED** to obtain an IP address via DHCP (from VirtualBox's Host-Only Network)
- **Configure GREEN** with a static IP address: `10.0.0.1` with subnet mask `255.255.255.0` (/24)
- **Enable DHCP Server on GREEN interface:** During EFW setup or via the web GUI after installation, configure the DHCP server for the GREEN zone (e.g., IP range 10.0.0.100 to 10.0.0.200)
- **Configure ORANGE** with a static IP address: `10.0.1.1` with subnet mask `255.255.255.0` (/24)
- **Enable DHCP Server on ORANGE interface:** Configure the DHCP server for the ORANGE zone (e.g., IP range 10.0.1.100 to 10.0.1.200)
- **Configure BLUE** with a static IP address: `10.0.2.1` with subnet mask `255.255.255.0` (/24)
- **Enable DHCP Server on BLUE interface:** Configure the DHCP server for the BLUE zone (e.g., IP range 10.0.2.100 to 10.0.2.200)
- Complete the installation. After installation, reboot and remove the ISO

### 1.3 Initial Network Configuration (Pre-Firewall)

#### On ServerVM, ClientVM, KaliClientVM, MobileClientVM:
- Initially, use the NAT adapter (Adapter 1) to perform `sudo apt update && sudo apt upgrade -y` to ensure all systems are up-to-date and have Python3 installed
- Once updates are done, shut down these VMs
- In VirtualBox settings for ServerVM, ClientVM, KaliClientVM and MobileClientVM, remove Network Adapter 1 (NAT). Only Network Adapter 2 (LAN_NET or BLUE_NET) should remain

#### On `FirewallVM` (Endian Firewall):
- After EFW installation and reboot, note the IP address of its **RED (WAN)** interface (from the console, it should be in your Host-Only network range, e.g., 192.168.56.X)
- The **GREEN (LAN)** interface should be `10.0.0.1`
- The **ORANGE (DMZ)** interface should be `10.0.1.1`
- The **BLUE (Wireless/VPN)** interface should be `10.0.2.1`

### 1.4 Network Configuration (Post-Firewall - Clients and Server)

#### On ServerVM (Ubuntu Server):
1. Start ServerVM
2. Identify the LAN_NET interface (e.g., `enp0s8`)
3. Edit its Netplan configuration (`/etc/netplan/*.yaml`)
4. Configure the interface to obtain an IP address via DHCP:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:  # Replace with your actual interface name
      dhcp4: yes  # Enable DHCP
      nameservers:
        addresses: [10.0.0.1]  # EFW GREEN IP as DNS resolver
```

5. Apply the changes: `sudo netplan apply`
6. **Important:** After applying, get the assigned IP address: `ip a show enp0s8`. This will be ServerVM_IP

#### On `ClientVM` (Ubuntu Desktop) and `KaliClientVM` (Kali Linux):
1. Start these VMs
2. Configure their `LAN_NET` interface (e.g., `enp0s8` or `eth1` depending on OS/version) to obtain an IP address via DHCP
3. Example Netplan configuration for the `LAN_NET` interface:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:  # Replace with your actual interface name
      dhcp4: yes  # Enable DHCP
      nameservers:
        addresses: [10.0.0.1]  # EFW GREEN IP as DNS resolver
```

4. Apply changes: `sudo netplan apply`
5. **Important:** After applying, get their assigned IP addresses: `ip a show enp0s8`. These will be `ClientVM_IP` and `KaliClientVM_IP`

#### On `MobileClientVM` (Ubuntu Desktop):
1. Start `MobileClientVM`
2. Identify the `BLUE_NET` interface (e.g., `enp0s8`)
3. Edit its Netplan configuration (`/etc/netplan/*.yaml`)
4. Configure the interface to obtain an IP address via DHCP:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:  # Replace with your actual interface name
      dhcp4: yes  # Enable DHCP
      nameservers:
        addresses: [10.0.2.1]  # EFW BLUE IP as DNS resolver
```

5. Apply the changes: `sudo netplan apply`
6. **Important:** After applying, get its assigned IP address: `ip a show enp0s8`. This will be `MobileClientVM_IP`

### 1.5 Connectivity Test (Through Firewall)

**Note:** For the following ping/traceroute tests, use the dynamically assigned IP addresses you obtained in Part 1.4 for ServerVM, ClientVM, KaliClientVM, and MobileClientVM.

#### From ClientVM:
- Ping ServerVM_IP (e.g., `ping 10.0.0.100`)
- Ping KaliClientVM_IP (e.g., `ping 10.0.0.101`)
- Ping MobileClientVM_IP (e.g., `ping 10.0.2.100`)
- Ping FirewallVM GREEN (`ping 10.0.0.1`)
- Ping FirewallVM BLUE (`ping 10.0.2.1`)
- Ping an external website (e.g., `ping google.com`). This tests internet access through EFW
- Run a traceroute to google.com to see the path through EFW

#### From ServerVM:
- Ping ClientVM_IP
- Ping KaliClientVM_IP
- Ping MobileClientVM_IP
- Ping FirewallVM GREEN (`ping 10.0.0.1`)
- Ping FirewallVM BLUE (`ping 10.0.2.1`)

#### From KaliClientVM:
- Ping ServerVM_IP
- Ping ClientVM_IP
- Ping MobileClientVM_IP
- Ping FirewallVM GREEN (`ping 10.0.0.1`)
- Ping FirewallVM BLUE (`ping 10.0.2.1`)

#### From MobileClientVM:
- Ping ServerVM_IP
- Ping ClientVM_IP
- Ping KaliClientVM_IP
- Ping FirewallVM GREEN (`ping 10.0.0.1`)
- Ping FirewallVM BLUE (`ping 10.0.2.1`)
- Ping an external website (e.g., `ping google.com`)

**Screenshot:** Include screenshots of successful pings and traceroute from ClientVM and MobileClientVM in your report.

#### Questions:
- **Question 1.1:** Explain why using Internal Networks for the LAN, DMZ, and Mobile sides, and a Host-Only Adapter for the WAN side of the firewall is suitable for this setup. How does this topology simulate a real-world network with segmented zones?
- **Question 1.2:** What is the purpose of the ORANGE (DMZ) zone in a firewall like Endian Firewall? How does it enhance network security compared to a simple two-zone (LAN/WAN) setup?
- **Question 1.3:** Describe the typical use case for the BLUE (Wireless/VPN) zone in a network environment. How does it logically separate mobile or VPN client traffic?
- **Question 1.4:** Explain the role of DHCP in this network setup. What are the advantages and disadvantages of using DHCP compared to static IP configuration for client machines in a lab environment?

---

## Part 2: Firewall Implementation (Endian Firewall Basic Configuration)

In this part, you will access the Endian Firewall web interface and configure basic firewall rules.

### 2.1 Access Endian Firewall Web GUI (from Host Machine)

1. Open a web browser on your host machine
2. Navigate to the Endian Firewall RED (WAN) IP address (the one you noted in Part 1.3, e.g., http://192.168.56.101)
3. Login with default credentials (usually admin/admin or admin/password initially, check EFW documentation if needed). You will be prompted to change the password during the first login
4. Complete the initial setup wizard (if it appears), ensuring correct time zone, DNS servers (e.g., 8.8.8.8, 8.8.4.4), and confirming interface assignments

### 2.2 Basic Firewall Rules Configuration

1. Navigate to Firewall > Traffic Rules (or similar section, EFW's GUI may vary slightly)
2. Endian Firewall typically has default rules allowing traffic from GREEN (LAN), ORANGE (DMZ), and BLUE (Wireless/VPN) zones to RED (WAN) and other zones

#### Create a Rule to Allow HTTP from ClientVM to ServerVM:
1. Click "Add a new rule"
2. Source Zone: GREEN
3. Source Address: Any (LAN_NET subnet) - Since ClientVM's IP is now dynamic
4. Destination Zone: GREEN
5. Destination Address: Any (LAN_NET subnet) - Since ServerVM's IP is now dynamic
6. Service: HTTP (TCP, Port 80)
7. Action: ALLOW
8. Description: "Allow HTTP from GREEN to GREEN"
9. Save and Apply Changes. Ensure this rule is processed before any general block rules

#### Create a Rule to Allow ICMP (Ping) between all Internal VMs (GREEN, ORANGE, BLUE):
1. Click "Add a new rule"
2. Source Zone: GREEN
3. Source Address: Any
4. Destination Zone: GREEN
5. Destination Address: Any
6. Service: ICMP (Ping)
7. Action: ALLOW
8. Description: "Allow ICMP on GREEN"
9. Save and Apply Changes

**Repeat for ORANGE and BLUE zones:** Create similar "Allow ICMP" rules for traffic originating from the ORANGE zone to any other zone, and from the BLUE zone to any other zone. This ensures basic ping connectivity for testing.

**Example for BLUE to GREEN ICMP:**
- Source Zone: BLUE
- Source Address: Any
- Destination Zone: GREEN
- Destination Address: Any
- Service: ICMP (Ping)
- Action: ALLOW
- Description: "Allow ICMP BLUE to GREEN"
- Save and Apply Changes

**Example for GREEN to BLUE ICMP:**
- Source Zone: GREEN
- Source Address: Any
- Destination Zone: BLUE
- Destination Address: Any
- Service: ICMP (Ping)
- Action: ALLOW
- Description: "Allow ICMP GREEN to BLUE"
- Save and Apply Changes

### 2.3 Testing Firewall Rules

#### From ClientVM:
- Verify you can still ping ServerVM_IP
- Verify you can access the web server on ServerVM using Lynx (`lynx http://ServerVM_IP/index.html`)

#### Demonstrate Blocking:
1. Go back to the Endian Firewall web GUI
2. Disable the "Allow HTTP from GREEN to GREEN" rule you just created (look for an enable/disable toggle or delete option). Apply Changes
3. From ClientVM: Try to access the web server again (`lynx http://ServerVM_IP/index.html`). It should now fail or time out
4. **Screenshot:** Include a screenshot of the failed web access after disabling the rule
5. Re-enable the rule and verify access is restored

#### Questions:
- **Question 2.1:** Explain the concept of a stateful firewall. How does Endian Firewall (or any stateful firewall) process outgoing and incoming traffic related to an established connection?
- **Question 2.2:** Describe the default behavior of Endian Firewall's GREEN zone (LAN) regarding outbound traffic to the RED zone (WAN). Why is this default important in a typical network setup?
- **Question 2.3:** Implement a rule to block HTTP traffic from the BLUE zone to the GREEN zone. Test this by trying to access the web server on ServerVM (`http://ServerVM_IP`) from MobileClientVM. Include a screenshot of the blocked attempt. Then, re-enable the traffic for the next parts.

---

## Part 3: Basic Web Server and Client (HTTP)

In this part, you will set up a simple web server on ServerVM and access it from ClientVM, KaliClientVM, and MobileClientVM. You will also implement a basic custom HTTP client.

### 3.1 Web Server Setup (on ServerVM)

1. Install Apache2 web server: `sudo apt update && sudo apt install apache2 -y`
2. Verify Apache is running: `sudo systemctl status apache2`
3. Create a simple HTML file:
   - `sudo nano /var/www/html/index.html`
   - Add content like:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to ServerVM</title>
</head>
<body>
    <h1>Hello from ServerVM!</h1>
    <p>This is a test web page for your Computer Networks assignment.</p>
    <p>Served by Apache on <SERVER_VM_DYNAMIC_IP_HERE></p>
</body>
</html>
```

4. Ensure Apache can read the file (permissions)

### 3.2 Web Client Access (from ClientVM, KaliClientVM, and MobileClientVM)

#### On ClientVM:
1. Install Lynx: `sudo apt install lynx -y`
2. Access your web page: `lynx http://ServerVM_IP/index.html`
3. **Screenshot:** Include a screenshot of the Lynx browser displaying your web page

#### On KaliClientVM:
1. Kali Linux usually has `curl` pre-installed
2. Access your web page: `curl http://ServerVM_IP/index.html`
3. **Screenshot:** Include a screenshot of the `curl` output displaying your web page

#### On MobileClientVM:
1. Install Lynx: `sudo apt install lynx -y`
2. Access your web page: `lynx http://ServerVM_IP/index.html`
3. **Screenshot:** Include a screenshot of the Lynx browser displaying your web page

**Question 3.1:** Describe the HTTP request and response messages exchanged when `lynx` or `curl` retrieves the `index.html` page. Focus on the key fields you would expect to see in the HTTP header.

### 3.3 Custom HTTP Client Implementation (on ClientVM)

Write a Python script (`http_client.py`) that acts as a basic HTTP client. It should:

1. Establish a TCP connection to ServerVM_IP on port 80
2. Send a simple HTTP GET request for `/index.html`
3. Receive and print the entire HTTP response (headers and body) from the server
4. Close the connection

**Hint:** Use Python's `socket` module.

```python
# Basic structure for http_client.py
import socket

# !!! IMPORTANT: Replace 'ServerVM_IP_HERE' with the actual IP address of ServerVM (obtained via DHCP)
SERVER_IP = 'ServerVM_IP_HERE'
SERVER_PORT = 80
REQUEST = b"GET /index.html HTTP/1.1\r\nHost: " + SERVER_IP.encode() + b"\r\nConnection: close\r\n\r\n"

try:
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.connect((SERVER_IP, SERVER_PORT))
    client_socket.sendall(REQUEST)

    response = b""
    while True:
        data = client_socket.recv(4096)
        if not data:
            break
        response += data
    print(response.decode('utf-8', errors='ignore')) # Decode for printing

except Exception as e:
    print(f"An error occurred: {e}")
finally:
    client_socket.close()
```

1. Run the script: `python3 http_client.py`
2. **Screenshot:** Include a screenshot of the script's output
3. **Deliverable:** Submit `http_client.py`

---

## Part 4: File Transfer Protocol (FTP)

You will set up an FTP server on ServerVM and use both a standard FTP client and a custom Python client to transfer files.

### 4.1 FTP Server Setup (on ServerVM)

1. Install vsftpd: `sudo apt install vsftpd -y`
2. Configure vsftpd for basic anonymous read access (for simplicity) or create a dedicated user. For this assignment, let's enable anonymous read access:
   - Edit `/etc/vsftpd.conf`:
     - `anonymous_enable=YES` (uncomment or add)
     - `local_enable=YES` (uncomment if you want local user access later)
     - `write_enable=YES` (uncomment if you want to allow anonymous uploads to `/var/ftp/pub`, but ensure security)
     - `anon_upload_enable=YES` (if `write_enable` is `YES`, allows anonymous uploads)
     - `anon_mkdir_write_enable=YES` (if `write_enable` is `YES`, allows anonymous directory creation)
   - Create a directory for anonymous uploads (if enabled): `sudo mkdir -p /var/ftp/pub && sudo chmod a+w /var/ftp/pub`
   - Restart vsftpd: `sudo systemctl restart vsftpd`
   - Place a test file in `/var/ftp` (e.g., `sudo nano /var/ftp/test_ftp.txt`)

### 4.2 Standard FTP Client Access (from ClientVM, KaliClientVM, or MobileClientVM)

#### On ClientVM:
1. Install FTP client: `sudo apt install ftp -y`
2. Connect to the FTP server: `ftp ServerVM_IP`
3. Login as anonymous (no password)
4. List files: `ls`
5. Download `test_ftp.txt`: `get test_ftp.txt`
6. Verify the file is downloaded on ClientVM
7. **Screenshot:** Include a screenshot of your FTP client session

#### On KaliClientVM:
1. Kali Linux typically has `ftp` client pre-installed
2. Repeat the steps above from KaliClientVM

#### On MobileClientVM:
1. Install FTP client: `sudo apt install ftp -y`
2. Repeat the steps above from MobileClientVM

**Question 4.1:** Explain the role of the two TCP connections in FTP (control and data). How do they differ in their purpose and lifecycle?

### 4.3 Custom FTP Client (LIST command) Implementation (on ClientVM)

Write a Python script (`ftp_client_list.py`) that connects to the FTP server and performs a LIST command. It should:

1. Establish a TCP control connection to ServerVM_IP on port 21
2. Receive the initial welcome message from the server
3. Send USER anonymous and PASS commands
4. Send the PASV command to initiate passive mode. Parse the server's response to get the IP and port for the data connection
5. Establish a new TCP data connection to the IP/port provided by the server
6. Send the LIST command over the control connection
7. Receive the directory listing over the data connection and print it
8. Close both connections

**Hint:** Parsing the `PASV` response can be tricky. It typically looks like `227 Entering Passive Mode (h1,h2,h3,h4,p1,p2)`. The IP is `h1.h2.h3.h4` and the port is `(p1 * 256) + p2`.

**Deliverable:** Submit `ftp_client_list.py`

---

## Part 5: Basic DNS Resolution

You will configure ClientVM, KaliClientVM, and MobileClientVM to use ServerVM for basic name resolution using the `/etc/hosts` file.

### 5.1 Configure ServerVM as a "DNS" Host (on ServerVM)

1. Edit ServerVM's `/etc/hosts` file: `sudo nano /etc/hosts`
2. Add an entry for a custom domain, e.g., `mywebserver.local`:

```
# IMPORTANT: Replace 10.0.0.10 with the actual dynamically assigned IP of ServerVM
<ServerVM_IP> mywebserver.local
```

3. Save and exit

### 5.2 Configure Clients to use ServerVM for DNS (on ClientVM, KaliClientVM, and MobileClientVM)

#### On ClientVM and KaliClientVM:
1. Edit their Netplan configuration (or `/etc/resolv.conf` if not using Netplan for DNS)
2. Set ServerVM_IP as the primary nameserver
3. Example Netplan configuration for the LAN_NET interface:

```yaml
# In /etc/netplan/*.yaml for enp0s8 (or similar)
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:  # Replace with your actual interface name
      dhcp4: yes
      nameservers:
        # IMPORTANT: Replace 10.0.0.10 with the actual dynamically assigned IP of ServerVM
        addresses: [<ServerVM_IP>]  # ServerVM's IP as DNS resolver
```

4. Apply the changes: `sudo netplan apply`

#### On MobileClientVM:
1. Edit its Netplan configuration
2. Set ServerVM_IP as the primary nameserver
3. Example Netplan configuration for the BLUE_NET interface:

```yaml
# In /etc/netplan/*.yaml for enp0s8 (or similar)
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:  # Replace with your actual interface name
      dhcp4: yes
      nameservers:
        # IMPORTANT: Replace 10.0.0.10 with the actual dynamically assigned IP of ServerVM
        addresses: [<ServerVM_IP>]  # ServerVM's IP as DNS resolver (reachable via EFW)
```

4. Apply the changes: `sudo netplan apply`

### 5.3 Test DNS Resolution (from ClientVM, KaliClientVM, and MobileClientVM)

#### On ClientVM:
1. Use ping or curl to test the custom domain:
   - `ping mywebserver.local`
   - `curl http://mywebserver.local/index.html`
2. Use dig or nslookup to explicitly query ServerVM for the custom domain:
   - `sudo apt install dnsutils -y` (if not installed)
   - `dig @ServerVM_IP mywebserver.local`
3. **Screenshot:** Include a screenshot showing successful resolution and access using the custom domain

#### On KaliClientVM:
1. Kali typically has `dig` and `nslookup` pre-installed
2. Repeat the ping, curl, and dig tests
3. **Screenshot:** Include a screenshot of the `dig` output

#### On MobileClientVM:
1. Install `dnsutils`: `sudo apt install dnsutils -y`
2. Repeat the ping, curl, and dig tests
3. **Screenshot:** Include a screenshot of the `dig` output

**Question 5.1:** Explain how DNS resolves a hostname to an IP address. Briefly describe the role of Authoritative DNS servers, TLD servers, and Root servers in a typical lookup process (even if not fully implemented in this basic setup).

---

## Part 6: Custom UDP Echo Client-Server

You will implement a simple UDP echo server on ServerVM and a corresponding client on ClientVM.

### 6.1 UDP Echo Server (on ServerVM)

Write a Python script (`udp_server.py`) that:

1. Creates a UDP socket
2. Binds the socket to ServerVM's LAN IP (the dynamically assigned IP you noted earlier) and a specific port (e.g., 12000)
3. Enters a loop to continuously receive UDP datagrams
4. When a datagram is received, it prints the message and the sender's address
5. Echoes the received message back to the sender

**Hint:** Use `socket.socket(socket.AF_INET, socket.SOCK_DGRAM)`, `bind()`, `recvfrom()`, `sendto()`.

**Deliverable:** Submit `udp_server.py`

### 6.2 UDP Echo Client (on ClientVM)

Write a Python script (`udp_client.py`) that:

1. Creates a UDP socket
2. Prompts the user to enter a message
3. Sends the message as a UDP datagram to ServerVM_IP and the server's port (12000)
4. Waits to receive an echo response from the server
5. Prints the echoed message

**Hint:** Use `socket.socket(socket.AF_INET, socket.SOCK_DGRAM)`, `sendto()`, `recvfrom()`.

**Deliverable:** Submit `udp_client.py`

### 6.3 Test UDP Echo

1. Start the UDP server on ServerVM: `python3 udp_server.py`
2. Run the UDP client on ClientVM: `python3 udp_client.py`
3. Send a few messages and observe the echo
4. **Screenshot:** Include a screenshot showing the client sending a message and receiving the echo, and the server's output

**Question 6.1:** Discuss the "connectionless" nature of UDP as demonstrated by your implementation. How does it differ from TCP in terms of setup and data exchange?

---

## Part 7: Custom TCP Echo Client-Server

You will implement a simple TCP echo server on ServerVM and a corresponding client on ClientVM.

### 7.1 TCP Echo Server (on ServerVM)

Write a Python script (`tcp_server.py`) that:

1. Creates a TCP socket
2. Binds the socket to ServerVM's LAN IP (the dynamically assigned IP you noted earlier) and a specific port (e.g., 12001)
3. Puts the socket into a listening state (`listen()`)
4. Enters a loop to continuously `accept()` new client connections
5. For each accepted connection:
   - Prints the client's address
   - Enters a sub-loop to receive data from this specific client
   - Echoes the received data back to the client
   - Breaks the sub-loop when the client closes its connection
   - Closes the client's connection socket

**Hint:** Use `socket.socket(socket.AF_INET, socket.SOCK_STREAM)`, `bind()`, `listen()`, `accept()`, `recv()`, `sendall()`.

**Deliverable:** Submit `tcp_server.py`

### 7.2 TCP Echo Client (on ClientVM)

Write a Python script (`tcp_client.py`) that:

1. Creates a TCP socket
2. Attempts to `connect()` to ServerVM_IP and the server's port (12001)
3. Prompts the user to enter messages
4. Sends messages to the server
5. Receives and prints the echoed messages
6. Allows the user to type 'quit' to close the connection

**Hint:** Use `socket.socket(socket.AF_INET, socket.SOCK_STREAM)`, `connect()`, `sendall()`, `recv()`.

**Deliverable:** Submit `tcp_client.py`

### 7.3 Test TCP Echo

1. Start the TCP server on ServerVM: `python3 tcp_server.py`
2. Run the TCP client on ClientVM: `python3 tcp_client.py`
3. Send a few messages and observe the echo
4. Type 'quit' to close the client. Observe the server's behavior
5. **Screenshot:** Include a screenshot showing the client sending messages and receiving echoes, and the server's output

#### Questions:
- **Question 7.1:** Describe the "connection-oriented" nature of TCP as demonstrated by your implementation. How does the server handle multiple clients compared to the UDP server?
- **Question 7.2:** Based on your experience with the UDP and TCP echo clients/servers, provide a scenario where UDP would be preferred over TCP, and vice-versa, justifying your choices based on their characteristics (reliability, connection setup, overhead).

---

## Submission Guidelines

- Compile your report as a single PDF document
- Organize your report clearly by part and section
- Ensure all specified screenshots are included and clearly labeled
- Provide clear explanations and answers to all questions
- Package all your Python source code files in a single .zip or .tar.gz archive
- Ensure your code is well-commented and executable

**Good luck with your assignment!** This hands-on experience will significantly deepen your understanding of computer network fundamentals.



