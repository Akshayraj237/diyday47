WireGuard VPN Setup Guide
This guide provides step-by-step instructions to configure your WireGuard VPN server on the GCP VM and connect your Windows laptop.

1. Preparation
Before starting, you need the External IP of your WireGuard VM. You can find this in the GCP Console under Compute Engine > VM Instances or by running terraform output (if configured).

Server External IP: <YOUR_VM_EXTERNAL_IP>
VPN Subnet: 10.100.0.0/24
Internal Network: 10.20.0.0/16
2. Server Configuration (GCP VM)
Step A: SSH into the VM
SSH into your WireGuard VM using the GCP Console button.

Step B: Install WireGuard
Run the following commands:

bash
sudo apt update
sudo apt install -y wireguard
Step C: Enable IP Forwarding
To allow the VM to act as a gateway to your cloud resources:

bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
Step D: Generate Server Keys
bash
sudo mkdir -p /etc/wireguard
cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
Note: Keep these keys handy. You will need the privatekey for the server config and publickey for the client config.

Step E: Create Server Configuration
Create /etc/wireguard/wg0.conf:

bash
sudo nano /etc/wireguard/wg0.conf
Paste the following (replace <SERVER_PRIVATE_KEY> with the content of your privatekey file):

ini
[Interface]
PrivateKey = <SERVER_PRIVATE_KEY>
Address = 10.100.0.1/24
ListenPort = 51820
# PostUp/PostDown rules for NAT (to access other VPC resources)
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o $(ip route show default | awk '{print $5}') -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o $(ip route show default | awk '{print $5}') -j MASQUERADE
# Client: Windows Laptop (We will add this after generating client keys)
# [Peer]
# PublicKey = <CLIENT_PUBLIC_KEY>
# AllowedIPs = 10.100.0.2/32
Step F: Start the VPN
bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
3. Client Configuration (Windows Laptop)
Step A: Open WireGuard App
Click Add Tunnel > Add empty tunnel...
Give it a name (e.g., GCP-VPN).
Step B: Configuration
A default configuration with a fresh PrivateKey and PublicKey will be generated. Copy the Public Key shown in the app.

Paste the following into the config block (replace placeholders):

ini
[Interface]
PrivateKey = <YOUR_WINDOWS_PRIVATE_KEY_ALREADY_THERE>
Address = 10.100.0.2/32
DNS = 1.1.1.1
[Peer]
PublicKey = <SERVER_PUBLIC_KEY_FROM_STEP_2D>
AllowedIPs = 10.100.0.0/24, 10.20.0.0/16, 172.16.0.16/28
Endpoint = <YOUR_VM_EXTERNAL_IP>:51820
PersistentKeepalive = 25
4. Finalizing the Connection
Add Client to Server
Back on the GCP VM, edit wg0.conf and add the Windows client's Public Key:

bash
sudo nano /etc/wireguard/wg0.conf
Add this at the end:

ini
[Peer]
PublicKey = <YOUR_WINDOWS_PUBLIC_KEY>
AllowedIPs = 10.100.0.2/32
Restart WireGuard on the server:

bash
sudo systemctl restart wg-quick@wg0
Connect
On your Windows laptop, click Activate in the WireGuard app.

6. Troubleshooting
Error: wg-quick: 'wg0' already exists
If the service fails to start with this error, it means the interface was not cleaned up properly. Fix:

bash
sudo wg-quick down wg0
sudo systemctl restart wg-quick@wg0
7. Verification
Try to ping an internal IP of a resource in your VPC (e.g., a GKE node or another VM):

cmd
ping 10.20.0.x