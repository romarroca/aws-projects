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



