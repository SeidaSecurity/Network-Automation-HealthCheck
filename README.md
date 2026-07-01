# Network-Automation-HealthCheck
A small Cisco Modeling Labs project to demonstrate network scripting and Wireshark packet analysis
By Justin Williams

#Overview
This project enabled me to practice network automation and packet analysis to monitor certain aspects of network devices in Cisco Modeling Labs (CML). A python script using the Netmiko Library makes SSH connections to each device and pulls interface status as well as neighbor device information using Cisco Discovery Protocol (CDP). Wireshark packet captures are then used to examine behavior between the devices when the script runs and see management traffic as it moves across the wire.

#Topology
-1 Router(R1) - connects to the switches and end host in the internal network in subnet 192.168.1.0/24. Its external interface connects to a CML External Connector, using NAT to enable access to the internet and allowing me to download the proper dependencies to write/run the script.
-2 Switches(SW1,SW2) - connects to the router and host node, configured with SVIs for SSH access.
-1 Host node(PC1) - used to write and run the python script, and its link connected to SW2 was used for packet capture.
<img width="949" height="450" alt="image" src="https://github.com/user-attachments/assets/9a65c0d2-b603-40ba-8e84-cf8a01ef973b" />


#Script
"health_check.py" loops through a local list of devices, connecting via SSH and doing the following:
1. Identifies the device from the CLI prompt
2. Runs the "show ip interface brief" command and flags interfaces not in an "up" state
3. Runs the "show cdp neighbors" command to confirm the devices neighbor relationships match the expected topology
4. Reports results on a per-device basis, continuing throught the list even if a connection fails

Some sample code written in VSCode before move the file into the host node, and subsequent results on the node:
<img width="777" height="678" alt="image" src="https://github.com/user-attachments/assets/536447ea-b307-445a-a7a4-136fe0a4230d" />
<img width="777" height="701" alt="image" src="https://github.com/user-attachments/assets/b5efcd9b-69cb-492b-8b5c-d1f696cbee6b" />

#Problems Encountered
-Modern SSH clients disable older algorithms like diffie-hellman-group14-sha1, which my Router using a older IOS image relied on. Resolved by allowing the required algorithms on ssh clients and within Netmiko's "disabled_algorithims" parameter.
-With my External Connector Node acting as a dhcp server, I noticed that dhcp clients were not recieving a dns server to resolve canonical names, so I resolved this by manually configuring Googles DNS server at 8.8.8.8 to resolve names.

#Packet Analysis
Capture was taken on the link between PC1 and SW2 while the script ran and then I examined the packets in Wireshark.
<img width="1915" height="1007" alt="image" src="https://github.com/user-attachments/assets/df09466e-d7d0-418b-a332-3f5d778906f0" />
The TCP handshake and establishment of the algorithim are visible in the capture, but after the key exchange the capture consist on encrypted application data which confirms that the commands that are being sent by the script
are protected in transit.
<img width="1913" height="1007" alt="image" src="https://github.com/user-attachments/assets/60443f8e-c7ed-44dd-bb9b-df5f95089c36" />
On the other hand, CDP traffic can be clearly seen in plaintext in the capture (Only one such packet was caught in the capture). This ehanced my understanding that CDP should only be enabled on trusted or isolated virtual networks as a matter of Security. CDP as a layer 2 protocol can be useful to understand connecting devices and map out topologies, but sends information about network devices is sent in plaintext so measures should be in place to protect this data. 

#Results
Running the script confirmed all expected interfaces reporting up, and administratively down interfaces were correctly flagged. Output also showed the intended topology using CDP, and traffic from the script was properly encrypted with SSH

#Skills Practiced
-Network Automation with Python and Netmiko
-Cisco IOS configuration: NAT/PAT, SSH hardening, CDP, Remote device management
-Packet capture and analysis with Wireshark
-IP connectivity troubleshooting
#Additional Notes
I will also attatch the YAML File to the repository if one wishes to recreate the lab.
*Internet connection via the CML External Connector will vary depending on your home network implementation, please look up Cisco documentation to properly configure the External Connector.

