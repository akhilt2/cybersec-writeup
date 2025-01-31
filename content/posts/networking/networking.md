+++
title = 'Task 4: Networking'
date = 2025-01-30T23:56:46+05:30
author = 'Akhil T'
weight = 2
draft = false
+++

To setup inter-site VPN connection, I have used pfSense as the VPN server running on qemu and two alpine linux VMs (alpine1 and alpine2) as clients which are also running in qemu. The network topology is such that the two alpine VMs are connected to the pfSense VM through a bridge network. The network topology is as shown below:

```
+---------------------+        +---------------------+        +---------------------+
|                     |        |                     |        |                     |
|      Alpine VM1     |        |      pfSense VM     |        |      Alpine VM2     |
|                     |        |                     |        |                     |
+---------------------+        +---------------------+        +---------------------+
        ^                            ^           ^                ^
        |                            |           |                |
        +----------------------------+           +----------------+
```

I am using WireGuard VPN protocol for setting up the VPN connection. WireGuard server is running on the pfSense VM and the other two alpine VMs are the peers connected to the VPN server. Thus both the to alpine VMs can communicate with each other through this VPN connection.

To setup this wireguard VPN, first we will install wireguard in the pfSense package manager. Then we will configure the tunnel by generating both public and private key and assigning an IP address to the pfSense which could be used in the VPN network. Then we will install wireguard in both the alpine instances. Create both private and public keys for both of the VMs. The private key we generated will be used only in the configuration file of that instance and the public key is shared to other instances to establish the connection. A configuration file is written in both the alpine instances. The one that is written in alpine2 is given below:
```
[Interface]
PrivateKey = CAfwrCTh7gm91HDdYZQZ6hNaXRuD+ME12emIotP3UVI=
Address = 192.168.200.3 

[Peer]
PublicKey = 2VPnRp80rudUfD7XwiDLUeEU9fRAVT0VUFlQ8RGFeis=
Endpoint = 192.168.122.24:51820
AllowedIPs = 192.168.200.0/24
PersistentKeepalive = 25
```
The private key in the config file is the one for alpine2 and the public key is the one to which bit needs to be connected which in this case is pfSense. The address is the IP address of the alpine2 in the VPN network. The endpoint is the IP address of the pfSense VM and the port number of the wireguard server. The allowed IPs is the range of IP addresses that are allowed to communicate with this instance. The persistent keepalive is the time interval in seconds between the keepalive packets.
After writing the configuration file, we will start the wireguard service in both the alpine instances. The connection is established between the alpine instances and the pfSense VM, thus enabling communication between both the alpine VMs. The connection is tested by pinging the IP address of the other alpine instance from one of the alpine instances. The connection is successful and the ping is successful.

The I asked ChatGPT to give a server and client code to communicate between two machines. The code for the server is given below:
```python
import socket

def start_server():
    host = '0.0.0.0'  # Listen to every interface
    port = 12345      # Some random port number

    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(1)
    print(f"Server is listening on {host}:{port}")

    conn, addr = server_socket.accept()
    print(f"Connection established with {addr}")

    while True:
        data = conn.recv(1024)
        if not data:
            break
        print(f"Received: {data.decode()}")
        conn.sendall(data)  # Send the received data back to the client

    conn.close()
    server_socket.close()

if __name__ == "__main__":
    start_server()
```
The code for the client is given below:
```python
import socket

def start_client():
    host = ''               # Server's IP
    port = 12345            # Same port as the server

    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.connect((host, port))
    print(f"Connected to server at {host}:{port}")

    try:
        while True:
            message = input("Enter message to send: ")
            if message.lower() == 'exit':
                break
            client_socket.sendall(message.encode())
            data = client_socket.recv(1024)
            print(f"Server replied: {data.decode()}")
    finally:
        client_socket.close()

if __name__ == "__main__":
    start_client()
```
I changed the host IP in the client code to the IP address of the alpine2 instance. I ran the server code in the alpine1 instance and the client code in the alpine2 instance. The connection was established. The message sent from the client is received by the server and the server sends back the same message to the client. Thus, connection and communication between the two machines is established successfully through the VPN network.

[Click here to view the Video Recording of the VPN connection and communication between the two alpine VMs](/cybersec-writeup/networking/pfSense.mp4)
