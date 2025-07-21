and MobileClientVM, remove Network Adapter 1 (NAT). Only Network Adapter 2 (LAN\_NET or BLUE\_NET) should remain.

  * **On `FirewallVM` (Endian Firewall):**
      * After EFW installation and reboot, note the IP address of its **RED (WAN)** interface (from the console, it should be in your Host-Only network range, e.g., 192.168.56.X).
      * The **GREEN (LAN)** interface should be `10.0.0.1`.
      * The **ORANGE (DMZ)** interface should be `10.0.1.1`.
      * The **BLUE (Wireless/VPN)** interface should be `10.0.2.1`.

1.4 Network Configuration (Post-Firewall - Clients and Server):

  * On ServerVM (Ubuntu Server):
    
      * Start ServerVM.
      * Identify the LAN\_NET interface (e.g., enp0s8).
      * Edit its Netplan configuration (/etc/netplan/\*.yaml).
      * Configure the interface to obtain an IP address via DHCP:
    
    <!-- end list -->

    ``` yaml
    network:
      version: 2
      renderer: networkd
      ethernets:
        enp0s8:  # Replace with your actual interface name
          dhcp4: yes  # Enable DHCP
          nameservers:
            addresses: [10.0.0.1]  # EFW GREEN IP as DNS resolver
    
    ```
    
      * Apply the changes: `sudo netplan apply`
      * **Important:** After applying, get the assigned IP address: `ip a show enp0s8`. This will be ServerVM\_IP.

  * **On `ClientVM` (Ubuntu Desktop) and `KaliClientVM` (Kali Linux):**
    
      * Start these VMs.
      * Configure their `LAN_NET` interface (e.g., `enp0s8` or `eth1` depending on OS/version) to obtain an IP address via DHCP.
      * Example Netplan configuration for the `LAN_NET` interface:
    
    <!-- end list -->

    ``` yaml
    network:
      version: 2
      renderer: networkd
      ethernets:
        enp0s8:  # Replace with your actual interface name
          dhcp4: yes  # Enable DHCP
          nameservers:
            addresses: [10.0.0.1]  # EFW GREEN IP as DNS resolver
    
    ```
    
      * Apply changes: `sudo netplan apply`
      * **Important:** After applying, get their assigned IP addresses: `ip a show enp0s8`. These will be `ClientVM_IP` and `KaliClientVM_IP`.

  * **On `MobileClientVM` (Ubuntu Desktop):**
    
      * Start `MobileClientVM`.
      * Identify the `BLUE_NET` interface (e.g., `enp0s8`).
      * Edit its Netplan configuration (`/etc/netplan/*.yaml`).
      * Configure the interface to obtain an IP address via DHCP:
    
    <!-- end list -->

    ``` yaml
    network:
      version: 2
      renderer: networkd
      ethernets:
        enp0s8:  # Replace with your actual interface name
          dhcp4: yes  # Enable DHCP
          nameservers:
            addresses: [10.0.2.1]  # EFW BLUE IP as DNS resolver
    
    ```
    
      * Apply the changes: `sudo netplan apply`
      * **Important:** After applying, get its assigned IP address: `ip a show enp0s8`. This will be `MobileClientVM_IP`.

1.5 Connectivity Test (Through Firewall):

  * Note: For the following ping/traceroute tests, use the dynamically assigned IP addresses you obtained in Part 1.4 for ServerVM, ClientVM, KaliClientVM, and MobileClientVM.

  * From ClientVM:
    
      * Ping ServerVM\_IP (e.g., ping 10.0.0.100).
      * Ping KaliClientVM\_IP (e.g., ping 10.0.0.101).
      * Ping MobileClientVM\_IP (e.g., ping 10.0.2.100).
      * Ping FirewallVM GREEN (ping 10.0.0.1).
      * Ping FirewallVM BLUE (ping 10.0.2.1).
      * Ping an external website (e.g., ping google.com). This tests internet access through EFW.
      * Run a traceroute to google.com to see the path through EFW.

  * From ServerVM:
    
      * Ping ClientVM\_IP.
      * Ping KaliClientVM\_IP.
      * Ping MobileClientVM\_IP.
      * Ping FirewallVM GREEN (ping 10.0.0.1).
      * Ping FirewallVM BLUE (ping 10.0.2.1).

  * From KaliClientVM:
    
      * Ping ServerVM\_IP.
      * Ping ClientVM\_IP.
      * Ping MobileClientVM\_IP.
      * Ping FirewallVM GREEN (ping 10.0.0.1).
      * Ping FirewallVM BLUE (ping 10.0.2.1).

  * From MobileClientVM:
    
      * Ping ServerVM\_IP.
      * Ping ClientVM\_IP.
      * Ping KaliClientVM\_IP.
      * Ping FirewallVM GREEN (ping 10.0.0.1).
      * Ping FirewallVM BLUE (ping 10.0.2.1).
      * Ping an external website (e.g., ping google.com).

  * Screenshot: Include screenshots of successful pings and traceroute from ClientVM and MobileClientVM in your report.

  * Question 1.1: Explain why using Internal Networks for the LAN, DMZ, and Mobile sides, and a Host-Only Adapter for the WAN side of the firewall is suitable for this setup. How does this topology simulate a real-world network with segmented zones?

  * Question 1.2: What is the purpose of the ORANGE (DMZ) zone in a firewall like Endian Firewall? How does it enhance network security compared to a simple two-zone (LAN/WAN) setup?

  * Question 1.3: Describe the typical use case for the BLUE (Wireless/VPN) zone in a network environment. How does it logically separate mobile or VPN client traffic?

  * Question 1.4: Explain the role of DHCP in this network setup. What are the advantages and disadvantages of using DHCP compared to static IP configuration for client machines in a lab environment?

**Part 2: Firewall Implementation (Endian Firewall Basic Configuration)**

In this part, you will access the Endian Firewall web interface and configure basic firewall rules.

2.1 Access Endian Firewall Web GUI (from Host Machine):

  * Open a web browser on your host machine.
  * Navigate to the Endian Firewall RED (WAN) IP address (the one you noted in Part 1.3, e.g., <http://192.168.56.101>).
  * Login with default credentials (usually admin/admin or admin/password initially, check EFW documentation if needed). You will be prompted to change the password during the first login.
  * Complete the initial setup wizard (if it appears), ensuring correct time zone, DNS servers (e.g., 8.8.8.8, 8.8.4.4), and confirming interface assignments.

2.2 Basic Firewall Rules Configuration:

  * Navigate to Firewall \> Traffic Rules (or similar section, EFW's GUI may vary slightly).

  * Endian Firewall typically has default rules allowing traffic from GREEN (LAN), ORANGE (DMZ), and BLUE (Wireless/VPN) zones to RED (WAN) and other zones.

  * Create a Rule to Allow HTTP from ClientVM to ServerVM:
    
      * Click "Add a new rule".
      * Source Zone: GREEN
      * Source Address: Any (LAN\_NET subnet) - Since ClientVM's IP is now dynamic
      * Destination Zone: GREEN
      * Destination Address: Any (LAN\_NET subnet) - Since ServerVM's IP is now dynamic
      * Service: HTTP (TCP, Port 80)
      * Action: ALLOW
      * Description: "Allow HTTP from GREEN to GREEN"
      * Save and Apply Changes. Ensure this rule is processed before any general block rules.

  * Create a Rule to Allow ICMP (Ping) between all Internal VMs (GREEN, ORANGE, BLUE):
    
      * Click "Add a new rule".
      * Source Zone: GREEN
      * Source Address: Any
      * Destination Zone: GREEN
      * Destination Address: Any
      * Service: ICMP (Ping)
      * Action: ALLOW
      * Description: "Allow ICMP on GREEN"

  - Save and Apply Changes.
  - Repeat for ORANGE and BLUE zones: Create similar "Allow ICMP" rules for traffic originating from the ORANGE zone to any other zone, and from the BLUE zone to any other zone. This ensures basic ping connectivity for testing.
  - Example for BLUE to GREEN ICMP:
      - Source Zone: BLUE
      - Source Address: Any
      - Destination Zone: GREEN
      - Destination Address: Any
      - Service: ICMP (Ping)
      - Action: ALLOW
      - Description: "Allow ICMP BLUE to GREEN"
      - Save and Apply Changes.
  - Example for GREEN to BLUE ICMP:
      - Source Zone: GREEN
      - Source Address: Any
      - Destination Zone: BLUE
      - Destination Address: Any
      - Service: ICMP (Ping)
      - Action: ALLOW
      - Description: "Allow ICMP GREEN to BLUE"
      - Save and Apply Changes.

2.3 Testing Firewall Rules:

  - From ClientVM:
      - Verify you can still ping ServerVM\_IP.
      - Verify you can access the web server on ServerVM using Lynx (`lynx http://ServerVM_IP/index.html`).
  - Demonstrate Blocking:
      - Go back to the Endian Firewall web GUI.
      - Disable the "Allow HTTP from GREEN to GREEN" rule you just created (look for an enable/disable toggle or delete option). Apply Changes.
      - From ClientVM: Try to access the web server again (`lynx http://ServerVM_IP/index.html`). It should now fail or time out.
      - Screenshot: Include a screenshot of the failed web access after disabling the rule.
      - Re-enable the rule and verify access is restored.
  - Question 2.1: Explain the concept of a stateful firewall. How does Endian Firewall (or any stateful firewall) process outgoing and incoming traffic related to an established connection?
  - Question 2.2: Describe the default behavior of Endian Firewall's GREEN zone (LAN) regarding outbound traffic to the RED zone (WAN). Why is this default important in a typical network setup?
  - Question 2.3: Implement a rule to block HTTP traffic from the BLUE zone to the GREEN zone. Test this by trying to access the web server on ServerVM (`http://ServerVM_IP`) from MobileClientVM. Include a screenshot of the blocked attempt. Then, re-enable the traffic for the next parts.

**Part 3: Basic Web Server and Client (HTTP)**

In this part, you will set up a simple web server on ServerVM and access it from ClientVM, KaliClientVM, and MobileClientVM. You will also implement a basic custom HTTP client.

3.1 Web Server Setup (on ServerVM):

  - Install Apache2 web server: `sudo apt update && sudo apt install apache2 -y`
  - Verify Apache is running: `sudo systemctl status apache2`
  - Create a simple HTML file:
      - `sudo nano /var/www/html/index.html`
      - Add content like:

<!-- end list -->

``` html
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

``` 
  -   Ensure Apache can read the file (permissions).

```

3.2 Web Client Access (from ClientVM, KaliClientVM, and MobileClientVM):

  - On ClientVM:
      - Install Lynx: `sudo apt install lynx -y`
      - Access your web page: `lynx http://ServerVM_IP/index.html`
      - Screenshot: Include a screenshot of the Lynx browser displaying your web page.
  - On KaliClientVM:
      - Kali Linux usually has `curl` pre-installed.
      - Access your web page: `curl http://ServerVM_IP/index.html`
      - Screenshot: Include a screenshot of the `curl` output displaying your web page.
  - On MobileClientVM:
      - Install Lynx: `sudo apt install lynx -y`
      - Access your web page: `lynx http://ServerVM_IP/index.html`
      - Screenshot: Include a screenshot of the Lynx browser displaying your web page.
  - Question 3.1: Describe the HTTP request and response messages exchanged when `lynx` or `curl` retrieves the `index.html` page. Focus on the key fields you would expect to see in the HTTP header.

3.3 Custom HTTP Client Implementation (on ClientVM):

  - Write a Python script (`http_client.py`) that acts as a basic HTTP client.
  - It should:
      - Establish a TCP connection to ServerVM\_IP on port 80.
      - Send a simple HTTP GET request for `/index.html`.
      - Receive and print the entire HTTP response (headers and body) from the server.
      - Close the connection.
  - **Hint:** Use Python's `socket` module.

<!-- end list -->

``` python
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

  - Run the script: `python3 http_client.py`
  - **Screenshot:** Include a screenshot of the script's output.
  - **Deliverable:** Submit `http_client.py`.

**Part 4: File Transfer Protocol (FTP)**

You will set up an FTP server on ServerVM and use both a standard FTP client and a custom Python client to transfer files.

4.1 FTP Server Setup (on ServerVM):

  - Install vsftpd: `sudo apt install vsftpd -y`
  - Configure vsftpd for basic anonymous read access (for simplicity) or create a dedicated user. For this assignment, let's enable anonymous read access:
      - Edit `/etc/vsftpd.conf`:
          - `anonymous_enable=YES` (uncomment or add)
          - `local_enable=YES` (uncomment if you want local user access later)
          - `write_enable=YES` (uncomment if you want to allow anonymous uploads to `/var/ftp/pub`, but ensure security)
          - `anon_upload_enable=YES` (if `write_enable` is `YES`, allows anonymous uploads)
          - `anon_mkdir_write_enable=YES` (if `write_enable` is `YES`, allows anonymous directory creation)
      - Create a directory for anonymous uploads (if enabled): `sudo mkdir -p /var/ftp/pub && sudo chmod a+w /var/ftp/pub`
      - Restart vsftpd: `sudo systemctl restart vsftpd`
      - Place a test file in `/var/ftp` (e.g., `sudo nano /var/ftp/test_ftp.txt`).

4.2 Standard FTP Client Access (from ClientVM, KaliClientVM, or MobileClientVM):

  - On ClientVM:
      - Install FTP client: `sudo apt install ftp -y`
      - Connect to the FTP server: `ftp ServerVM_IP`
      - Login as anonymous (no password).
      - List files: `ls`
      - Download `test_ftp.txt`: `get test_ftp.txt`
      - Verify the file is downloaded on ClientVM.
      - Screenshot: Include a screenshot of your FTP client session.
  - On KaliClientVM:
      - Kali Linux typically has `ftp` client pre-installed.
      - Repeat the steps above from KaliClientVM.
  - On MobileClientVM:
      - Install FTP client: \`sudo install ftp -y\`
\- Repeat the steps above from MobileClientVM.

  - Question 4.1: Explain the role of the two TCP connections in FTP (control and data). How do they differ in their purpose and lifecycle?

4.3 Custom FTP Client (LIST command) Implementation (on ClientVM):

  - Write a Python script (\`ftp\_client\_list.py\`) that connects to the FTP server and performs a LIST command.
  - It should:
      - Establish a TCP control connection to ServerVM\_IP on port 21.
      - Receive the initial welcome message from the server.
      - Send USER anonymous and PASS commands.
      - Send the PASV command to initiate passive mode. Parse the server's response to get the IP and port for the data connection.
      - Establish a new TCP data connection to the IP/port provided by the server.
      - Send the LIST command over the control connection.
      - Receive the directory listing over the data connection and print it.
      - Close both connections.
  - **Hint:** Parsing the \`PASV\` response can be tricky. It typically looks like \`227 Entering Passive Mode (h1,h2,h3,h4,p1,p2)\`. The IP is \`h1.h2.h3.h4\` and the port is \`(p1 \* 256) + p2\`.
  - **Deliverable:** Submit \`ftp\_client\_list.py\`.

**Part 5: Basic DNS Resolution**

You will configure ClientVM, KaliClientVM, and MobileClientVM to use ServerVM for basic name resolution using the /etc/hosts file.

5.1 Configure ServerVM as a "DNS" Host (on ServerVM):

  - Edit ServerVM's /etc/hosts file: \`sudo nano /etc/hosts\`
  - Add an entry for a custom domain, e.g., \`mywebserver.local\`:

\<\!-- end list --\>

``` 
# IMPORTANT: Replace 10.0.0.10 with the actual dynamically assigned IP of ServerVM
<ServerVM_IP> mywebserver.local

```

  - Save and exit.

5.2 Configure Clients to use ServerVM for DNS (on ClientVM, KaliClientVM, and MobileClientVM):

  - On ClientVM and KaliClientVM:
      - Edit their Netplan configuration (or /etc/resolv.conf if not using Netplan for DNS).
      - Set ServerVM\_IP as the primary nameserver.
      - Example Netplan configuration for the LAN\_NET interface:

\<\!-- end list --\>

``` 
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

``` 
  - Apply the changes: \`sudo netplan apply\`

```

  - On MobileClientVM:
      - Edit its Netplan configuration.
      - Set ServerVM\_IP as the primary nameserver.
      - Example Netplan configuration for the BLUE\_NET interface:

\<\!-- end list --\>

``` 
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

``` 
  - Apply the changes: \`sudo netplan apply\`

```

5.3 Test DNS Resolution (from ClientVM, KaliClientVM, and MobileClientVM):

  - On ClientVM:
      - Use ping or curl to test the custom domain:
          - \`ping mywebserver.local\`
          - \`curl http://mywebserver.local/index.html\`
      - Use dig or nslookup to explicitly query ServerVM for the custom domain:
          - \`sudo apt install dnsutils -y\` (if not installed)
          - \`dig @ServerVM\_IP mywebserver.local\`
      - Screenshot: Include a screenshot showing successful resolution and access using the custom domain.
  - On KaliClientVM:
      - Kali typically has \`dig\` and \`nslookup\` pre-installed.
      - Repeat the ping, curl, and dig tests.
      - Screenshot: Include a screenshot of the \`dig\` output.
  - On MobileClientVM:
      - Install \`dnsutils\`: \`sudo apt install dnsutils -y\`
      - Repeat the ping, curl, and dig tests.
      - Screenshot: Include a screenshot of the \`dig\` output.
  - Question 5.1: Explain how DNS resolves a hostname to an IP address. Briefly describe the role of Authoritative DNS servers, TLD servers, and Root servers in a typical lookup process (even if not fully implemented in this basic setup).

**Part 6: Custom UDP Echo Client-Server**

You will implement a simple UDP echo server on ServerVM and a corresponding client on ClientVM.

6.1 UDP Echo Server (on ServerVM):

  - Write a Python script (\`udp\_server.py\`) that:
      - Creates a UDP socket.
      - Binds the socket to ServerVM's LAN IP (the dynamically assigned IP you noted earlier) and a specific port (e.g., 12000).
      - Enters a loop to continuously receive UDP datagrams.
      - When a datagram is received, it prints the message and the sender's address.
      - Echoes the received message back to the sender.
  - **Hint:** Use \`socket.socket(socket.AF\_INET, socket.SOCK\_DGRAM)\`, \`bind()\`, \`recvfrom()\`, \`sendto()\`.
  - **Deliverable:** Submit \`udp\_server.py\`.

6.2 UDP Echo Client (on ClientVM):

  - Write a Python script (\`udp\_client.py\`) that:
      - Creates a UDP socket.
      - Prompts the user to enter a message.
      - Sends the message as a UDP datagram to ServerVM\_IP and the server's port (12000).
      - Waits to receive an echo response from the server.
      - Prints the echoed message.
  - **Hint:** Use \`socket.socket(socket.AF\_INET, socket.SOCK\_DGRAM)\`, \`sendto()\`, \`recvfrom()\`.
  - **Deliverable:** Submit \`udp\_client.py\`.

6.3 Test UDP Echo:

  - Start the UDP server on ServerVM: \`python3 udp\_server.py\`
  - Run the UDP client on ClientVM: \`python3 udp\_client.py\`
  - Send a few messages and observe the echo.
  - Screenshot: Include a screenshot showing the client sending a message and receiving the echo, and the server's output.
  - Question 6.1: Discuss the "connectionless" nature of UDP as demonstrated by your implementation. How does it differ from TCP in terms of setup and data exchange?

**Part 7: Custom TCP Echo Client-Server**

You will implement a simple TCP echo server on ServerVM and a corresponding client on ClientVM.

7.1 TCP Echo Server (on ServerVM):

  - Write a Python script (\`tcp\_server.py\`) that:
      - Creates a TCP socket.
      - Binds the socket to ServerVM's LAN IP (the dynamically assigned IP you noted earlier) and a specific port (e.g., 12001).
      - Puts the socket into a listening state (\`listen()\`).
      - Enters a loop to continuously \`accept()\` new client connections.
      - For each accepted connection:
          - Prints the client's address.
          - Enters a sub-loop to receive data from this specific client.
          - Echoes the received data back to the client.
          - Breaks the sub-loop when the client closes its connection.
          - Closes the client's connection socket.
  - **Hint:** Use \`socket.socket(socket.AF\_INET, socket.SOCK\_STREAM)\`, \`bind()\`, \`listen()\`, \`accept()\`, \`recv()\`, \`sendall()\`.
  - **Deliverable:** Submit \`tcp\_server.py\`.

7.2 TCP Echo Client (on ClientVM):

  - Write a Python script (\`tcp\_client.py\`) that:
      - Creates a TCP socket.
      - Attempts to \`connect()\` to ServerVM\_IP and the server's port (12001).
      - Prompts the user to enter messages.
      - Sends messages to the server.
      - Receives and prints the echoed messages.
      - Allows the user to type 'quit' to close the connection.
  - **Hint:** Use \`socket.socket(socket.AF\_INET, socket.SOCK\_STREAM)\`, \`connect()\`, \`sendall()\`, \`recv()\`.
  - **Deliverable:** Submit \`tcp\_client.py\`.

7.3 Test TCP Echo:

  - Start the TCP server on ServerVM: \`python3 tcp\_server.py\`
  - Run the TCP client on ClientVM: \`python3 tcp\_client.py\`
  - Send a few messages and observe the echo.
  - Type 'quit' to close the client. Observe the server's behavior.
  - Screenshot: Include a screenshot showing the client sending messages and receiving echoes, and the server's output.
  - Question 7.1: Describe the "connection-oriented" nature of TCP as demonstrated by your implementation. How does the server handle multiple clients compared to the UDP server?
  - Question 7.2: Based on your experience with the UDP and TCP echo clients/servers, provide a scenario where UDP would be preferred over TCP, and vice-versa, justifying your choices based on their characteristics (reliability, connection setup, overhead).

**Submission Guidelines:**

  - Compile your report as a single PDF document.
  - Organize your report clearly by part and section.
  - Ensure all specified screenshots are included and clearly labeled.
  - Provide clear explanations and answers to all questions.
  - Package all your Python source code files in a single .zip or .tar.gz archive.
  - Ensure your code is well-commented and executable.

Good luck with your assignment\! This hands-on experience will significantly deepen your understanding of computer network fundamentals.



