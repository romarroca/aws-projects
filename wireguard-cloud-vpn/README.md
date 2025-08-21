# WireGuard Cloud VPN on AWS

This project demonstrates how to build a secure VPN hub using WireGuard on an AWS EC2 instance.  
The VPN server acts as a cloud-based gateway, allowing remote devices and networks (e.g., my GL.iNet router) to connect securely without requiring a public IP at home.

## Features
- WireGuard server hosted on AWS (Ubuntu EC2 instance)
- Supports router and mobile clients
- Extensible for learning AWS networking and security concepts

##  1. AWS EC2 Setup
1. Launch an **Ubuntu EC2 instance** (t2.micro is enough).  
   - Assign a public IPv4.  
   - Open security group rules:
     - UDP `51820` (WireGuard) # This will be our listening port for our wireguard server
     - TCP `22` for SSH. # For managing our device remotely
     <img width="1218" height="1139" alt="image" src="https://github.com/user-attachments/assets/190b849b-6978-4857-8ae2-af96e1ce9ff4" />

     <img width="1697" height="526" alt="image" src="https://github.com/user-attachments/assets/c4bd4cf8-877a-4858-9cc8-2046b5d078e7" />


##  2. Install WireGuard: SSH using the generated key


<img width="1683" height="891" alt="image" src="https://github.com/user-attachments/assets/d9cfcd7f-1536-4cef-9140-f693b4e99ba5" />
   
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install wireguard -y
   ```

##  3. Generate Wireguard keys

   ```bash
   umask 077
   wg genkey | tee server_private.key | wg pubkey > server_public.key
   wg genkey | tee router_private.key | wg pubkey > router_public.key
   wg genkey | tee mobile_private.key | wg pubkey > mobile_public.key
   ```

##  4. Configure Wireguard Server

   ```bash
   [Interface]
   Address = 10.0.0.1/24
   SaveConfig = false
   PostUp = sysctl -w net.ipv4.ip_forward=1; iptables -A FORWARD -i wg0 -o wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -i wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT
   PostDown = iptables -D FORWARD -i wg0 -o wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -i wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT
   ListenPort = 51820
   PrivateKey = <server_private.key>
   
   [Peer]
   PublicKey = <router_public.key>
   AllowedIPs = 10.0.0.2/32, 192.168.8.0/24
   Endpoint = 72.179.178.119:37512
   PersistentKeepalive = 25
   
   [Peer]
   PublicKey = <mobile_public.key>
   AllowedIPs = 10.0.0.3/32
   PersistentKeepalive = 25
   ```

##  5. Start Wireguard server

   ```bash
   sudo systemctl enable wg-quick@wg0
   sudo systemctl start wg-quick@wg0
   sudo systemctl status wg-quick@wg0
   ```
##  6. Configure GL.iNet Router as Client

   <img width="582" height="807" alt="image" src="https://github.com/user-attachments/assets/5a174eb9-0fdb-43e6-9e16-75f56f44bda1" />

   <img width="1042" height="875" alt="image" src="https://github.com/user-attachments/assets/42610b20-6360-4e15-9ce5-fe51215fbf96" />

   <img width="596" height="401" alt="image" src="https://github.com/user-attachments/assets/8d01a8c0-d3db-4157-8b54-4130a8566185" />

   The keys here came from the genkey I generated from the EC2 instance.
   
## 7. On your remote device, in my example I used an android Device. Install Wireguard from the playstore and add the following config

   <img width="711" height="293" alt="image" src="https://github.com/user-attachments/assets/1c7a9ce0-adbd-43b8-99f5-60331ae2d48b" />

   The keys here came from the genkey I generated from the EC2 instance.
   
## 8. Testing and Verification

   - Verify first that wireguard is running on the EC2 instance

      ```bash
      sudo wg show
      ```
   <img width="928" height="514" alt="image" src="https://github.com/user-attachments/assets/3b5b5f53-46ff-40a2-b0bd-862e9a6cf65e" />

   - Enable the Wireguard client from your Gl-Inet Router by toggling the on/off button

     <img width="1116" height="870" alt="image" src="https://github.com/user-attachments/assets/7d986acc-652b-467d-a844-a1b207f1b054" />

   - Once enabled, host behind the router should be able to reach ec2 instance wireguard IP which is 10.0.0.1 but still be able to ping 8.8.8.8 as this setup is configured only for split-tunnel

      <img width="1075" height="881" alt="image" src="https://github.com/user-attachments/assets/98b09951-5810-47ad-97a6-e927051ef742" />


   - Now connect your mobile by enabling toggling the on/off button

     <img width="406" height="615" alt="image" src="https://github.com/user-attachments/assets/368bee21-9457-45c7-b71c-8bccd4650a62" />

   - From my mobile which is currently using mobile data and connected to my EC2 vpn hub, I can reach a subnet/host behind my router at home

     <img width="359" height="716" alt="image" src="https://github.com/user-attachments/assets/38a1a5e2-a9b5-49ab-81b6-c0a8721d308e" />

   - Verify conected clients from EC2 instance

     <img width="913" height="586" alt="image" src="https://github.com/user-attachments/assets/5d228627-378b-4969-a67d-6c35e310c88a" />

