# DHCP Server Setup on Ubuntu with LAN Client

This repository provides step-by-step documentation to set up a \*\*DHCP server using \*\*\`\` on Ubuntu. The server provides IP addresses to LAN-connected client machines.

---

## 📚 Table of Contents

```bash
- [System Roles and IP Plan](#system-roles-and-ip-plan)
- [Installation](#installation)
- [DHCP Configuration](#dhcp-configuration)
- [Netplan Static IP Configuration](#netplan-static-ip-configuration)
- [Client Configuration](#client-configuration)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Conclusion](#conclusion)
```

---

## 🔹 System Roles and IP Plan

* **Server (Ubuntu)**: 192.168.3.5 (LAN-facing interface: `enp2s0`)
* **Client (Ubuntu)**: DHCP client via LAN cable (interface: `enp37s0`)
* **Network**: 192.168.3.0/24 (static IP for server, dynamic range for clients)

Ensure both systems are connected via LAN and server-side interface is `UP` and `CONNECTED`.

---

## 👉 Installation

```bash
sudo apt update
sudo apt install isc-dhcp-server
```

Enable the service:

```bash
sudo systemctl enable isc-dhcp-server
```

---

## 📂 DHCP Configuration

Edit the file `/etc/dhcp/dhcpd.conf`:

```conf
default-lease-time 600;
max-lease-time 7200;
ddns-update-style none;
authoritative;

subnet 192.168.3.0 netmask 255.255.255.0 {
  range 192.168.3.100 192.168.3.200;
  option routers 192.168.3.5;
  option domain-name-servers 8.8.8.8;
  option domain-name "mcci.io";
}
```

Edit `/etc/default/isc-dhcp-server` and set:

```bash
INTERFACESv4="enp2s0"
```

Restart service:

```bash
sudo systemctl restart isc-dhcp-server
```

Check status:

```bash
sudo systemctl status isc-dhcp-server
```

---

## 🚧 Netplan Static IP Configuration (Server)

If using Netplan, configure static IP for `enp2s0`:

`/etc/netplan/01-dhcp-server.yaml`

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp2s0:
      dhcp4: no
      addresses: [192.168.3.5/24]
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

Apply changes:

```bash
sudo netplan apply
```

Alternatively, if Netplan is not used, configure with `ifconfig`:

```bash
sudo ip addr add 192.168.3.5/24 dev enp2s0
sudo ip link set enp2s0 up
```

---

## 🤖 Client Configuration (Ubuntu)

Make sure the client is connected via LAN to the server.

Release old IP (if any):

```bash
sudo dhclient -r enp37s0
```

Request new IP from DHCP server:

```bash
sudo dhclient enp37s0
```

Check IP:

```bash
ip a show enp37s0
```

Expected: IP from `192.168.3.100 - 192.168.3.200`.

---

## 🔍 Testing

On the **server**, view assigned leases:

```bash
cat /var/lib/dhcp/dhcpd.leases
```

Live log:

```bash
sudo journalctl -f -u isc-dhcp-server
```

On the **client**, test connectivity:

```bash
ping 192.168.3.5
```

---

## ⚠️ Troubleshooting

* **Client not getting IP**: Ensure server interface is `UP` and cable is connected (check `ip a`).
* **Only DHCPDISCOVER seen**: Server not responding. Verify `dhcpd.conf` and interface binding.
* **"Network unreachable"**: Client not in correct subnet. Check cables and link status.
* **Multiple interfaces warning**: Bind only one interface in `/etc/default/isc-dhcp-server`.

---

## 🔝 Conclusion

You now have a functioning **DHCP server** on Ubuntu, dynamically assigning IP addresses to LAN clients. Ideal for internal lab, testing, or isolated networks.

Feel free to fork and customize this setup for more advanced use cases like PXE boot or VLAN-based DHCP.
