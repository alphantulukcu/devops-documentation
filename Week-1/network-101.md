# Network 101

![alt text](../screenshots/network.png)

In the third day of CodeCamp’24, I explored a broad range of essential networking concepts, tools, and protocols that form the backbone of modern digital communication. Here's a detailed summary of what I learned:

## What is a Network?
A network is a system where various network devices, such as computers and routers, communicate and share resources over communication channels, either within a localized environment or over vast distances.

## Network Topologies

> **LAN (Local Area Network):** A network confined to a small area, like within a single building.

> **MAN (Metropolitan Area Network):** A larger network that spans a city or a metropolitan area.

> **WAN (Wide Area Network):** An expansive network that covers large geographical areas, often linking different cities or countries.

> **OSI Reference Model:**
The OSI (Open Systems Interconnection) model is a conceptual framework used to understand and implement network communication in seven distinct layers. Each layer serves a specific function and communicates with the layers directly above and below it.

### Physical Layer (Layer 1)

This layer handles the physical connection between devices, transmitting data as electrical, light, or radio signals. Devices like hubs operate at this layer, converting binary data (ones and zeros) into these signals and vice versa.

### Data Link Layer (Layer 2)

The Data Link Layer manages the protocol for data transfer between devices on the same network. It is divided into two sublayers:

> **MAC (Media Access Control):** Responsible for the physical addressing (using MAC addresses) and error detection using CRC (Cyclic Redundancy Check).

> **LLC (Logical Link Control):** Ensures error correction, flow control, and the proper routing of data frames to the network layer.

### Network Layer (Layer 3)

This layer is responsible for logical addressing and routing. It ensures that data packets are correctly routed from the source to the destination across different networks using IP addresses. Routers operate at this layer to determine the best path for data transmission.

> **IP Address:** A unique identifier assigned to each device on a network, facilitating communication between devices.

> **IPv4:** The current standard, using 32-bit addresses, allowing for approximately 4.3 billion unique addresses.

> **IPv6:** The newer standard, using 128-bit addresses, vastly increasing the number of available addresses.

> **Subnet Mask:** A 32-bit number that divides the IP address into the network and host portions, allowing for the creation of smaller sub-networks within a larger network.

> **Gateway:** A network device, often a router, that acts as an access point or "gateway" to another network, such as the internet.

### Transport Layer (Layer 4)
The Transport Layer is responsible for delivering data across network connections, ensuring that it arrives intact and in order. It uses protocols like:

> **TCP (Transmission Control Protocol):** Establishes a reliable connection between two devices using a process known as the three-way handshake (SYN, SYN-ACK, ACK).

> **UDP (User Datagram Protocol):** A connectionless protocol used for fast, but less reliable, transmission, often used in real-time applications like video streaming.

> **ICMP (Internet Control Message Protocol):** Used for error reporting and network diagnostics, such as ping commands.

Port Numbers
Port numbers identify specific processes or services on a device, enabling multiple services to run simultaneously on the same IP address such as Well-Known Ports (0-1024), Registered Ports (1025-49151), Dynamic Ports (49152-65535)

### Higher OSI Layers
**Session Layer (Layer 5):** Manages and controls the connections between computers, ensuring that data streams are properly coordinated and managed.

**Presentation Layer (Layer 6):** Translates data between the application layer and the network, ensuring that data sent from the application layer of one system is readable by the application layer of another. It handles data encryption and compression.

**Application Layer (Layer 7):** The closest layer to the end-user, it interfaces with software applications to implement communication protocols like HTTP, FTP, DNS, etc.
Domain Name System (DNS)
DNS is a critical service that translates human-friendly domain names (like www.example.com) into IP addresses, allowing users to access websites without needing to memorize numeric IP addresses.

**SSH:** A secure protocol for remote access and management.
**Telnet:** A protocol used for remote communication but less secure than SSH.
**TCPDUMP:** A powerful command-line packet analyzer.
**Netcat:** A versatile networking utility used for reading from and writing to network connections.
**Netstat:** A command-line tool that displays network connections and routing tables.
**IPTABLES:** A command-line utility for configuring network packet filtering rules on Linux.
**DNS Tools:** Commands and utilities for querying and managing DNS records.

This week provided a comprehensive introduction to networking, covering the essential concepts and tools needed to understand and work with computer networks effectively.

## Cases

In the cases, we learned how to use `Cisco Packet Tracer`. We handle some network scenarios and made some hands-on with the mentor. For example, the first case is as follows:

> Creare two servers with `192.168.1.10` and `192.168.2.10` ip addresses, and ensure both devices ping each other without any additional devices.

