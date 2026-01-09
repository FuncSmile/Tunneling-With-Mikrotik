# Tunneling with MikroTik

A comprehensive guide and toolkit for setting up secure tunneling solutions using MikroTik routers.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Architecture](#architecture)
- [Installation](#installation)
- [Server Configuration](#server-configuration)
- [Client Configuration](#client-configuration)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This project provides scripts, configurations, and documentation for establishing secure tunneling connections through MikroTik RouterOS devices. Whether you need to create VPN tunnels, secure remote access, or interconnect multiple networks, this toolkit offers practical solutions.

![Architecture Diagram](https://via.placeholder.com/800x400?text=MikroTik+Tunneling+Architecture)

## ✨ Features

- **Easy Configuration**: Simple-to-use scripts for quick setup
- **Secure Communication**: Encrypted tunnel protocols
- **Flexible Deployment**: Support for various tunnel types and scenarios
- **Server & Client Separation**: Dedicated scripts for each role
- **Well-Documented**: Comprehensive configuration guides
- **Scalable**: Suitable for small to enterprise deployments

## 📋 Prerequisites

Before getting started, ensure you have:

- **MikroTik Router**: RouterOS version 6.48+ recommended
- **Network Access**: Administrative access to your MikroTik device
- **Basic Networking Knowledge**: Understanding of routing and tunneling concepts
- **Required Tools**:
  - SSH client or Winbox (for management)
  - Terminal/Command prompt
  - Text editor for script modifications

## 🏗️ Architecture

### System Design

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Network                            │
│  ┌──────────────────────────────────────────────────────┐   │
│  │         Client MikroTik Router                        │   │
│  │  - VPN Client Configuration                          │   │
│  │  - Tunnel Encryption                                │   │
│  │  - Route Management                                 │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
              │
              │ Encrypted Tunnel
              │
┌─────────────────────────────────────────────────────────────┐
│                    Server Network                            │
│  ┌──────────────────────────────────────────────────────┐   │
│  │         Server MikroTik Router                        │   │
��  │  - VPN Server Configuration                          │   │
│  │  - Tunnel Termination                               │   │
│  │  - Traffic Management                               │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Network Flow Diagram

![Network Flow](https://via.placeholder.com/800x300?text=Network+Flow+Diagram)

## 💻 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/FuncSmile/Tunneling-With-Mikrotik.git
cd Tunneling-With-Mikrotik
```

### 2. File Structure

```
Tunneling-With-Mikrotik/
├── README.md
├── server/
│   ├── setup-server.rsc
│   ├── config/
│   │   ├── firewall-rules.rsc
│   │   ├── nat-rules.rsc
│   │   └── interface-config.rsc
│   └── README.md
├── client/
│   ├── setup-client.rsc
│   ├── config/
│   │   ├── tunnel-config.rsc
│   │   ├── routing-rules.rsc
│   │   └── security-config.rsc
│   └── README.md
├── examples/
│   ├── basic-tunnel-setup.md
│   ├── advanced-scenarios.md
│   └── troubleshooting-guide.md
└── docs/
    ├── protocol-reference.md
    └── configuration-guide.md
```

### 3. Initial Setup

Navigate to the appropriate section (Server or Client) based on your role.

---

## 🖥️ Server Configuration

### Overview

The server component handles tunnel termination, traffic routing, and security policies for incoming client connections.

![Server Setup](https://via.placeholder.com/600x400?text=Server+Configuration+Setup)

### Quick Start

**Location**: `server/setup-server.rsc`

```bash
# 1. Connect to your Server MikroTik Router via SSH or Winbox
ssh admin@<server-ip>

# 2. Upload and execute the server setup script
/system script run setup-server

# 3. Verify the configuration
/interface print
```

### Configuration Files

#### Main Server Script: `server/setup-server.rsc`

This script performs the following:
- Initializes tunnel interfaces
- Configures IP addressing
- Sets up basic routing
- Applies security rules

```routeros
# Example snippet - Full script available in server/setup-server.rsc

# Create tunnel interface
/interface/ovpn-server/server/set enabled=yes

# Configure IP Pool
/ip/pool/add name="tunnel_pool" ranges=10.0.0.2-10.0.0.254

# Add bridge interface
/interface/bridge/add name=bridge_tunnel
```

#### Firewall Configuration: `server/config/firewall-rules.rsc`

Manages incoming and outgoing traffic rules:
- Whitelist trusted clients
- Filter malicious traffic
- Rate limiting configurations

```routeros
# Accept established connections
/ip/firewall/filter/add chain=input \
    connection-state=established,related action=accept

# Drop invalid traffic
/ip/firewall/filter/add chain=input \
    connection-state=invalid action=drop
```

#### NAT Configuration: `server/config/nat-rules.rsc`

Handles Network Address Translation:
- Source NAT for server traffic
- Destination NAT for port forwarding
- Connection tracking

```routeros
# Source NAT for tunnel traffic
/ip/firewall/nat/add chain=srcnat \
    out-interface=ether1 action=masquerade
```

#### Interface Configuration: `server/config/interface-config.rsc`

Sets up physical and virtual interfaces:
- IP address assignments
- MTU configuration
- Bandwidth management

```routeros
# Configure tunnel interface IP
/ip/address/add address=10.0.0.1/24 interface=tunnel_interface
```

### Server Setup Instructions

1. **Access Your Router**
   ```bash
   ssh admin@<your-server-ip>
   ```

2. **Load Configuration Scripts**
   ```bash
   /import file-name=server/setup-server.rsc
   ```

3. **Verify Installation**
   - Check tunnel interfaces: `/interface print`
   - Verify IP configuration: `/ip/address print`
   - Test connectivity: `/ping <client-ip>`

4. **Monitor Activity**
   ```bash
   /log print
   /interface monitor-traffic ether1
   ```

---

## 👥 Client Configuration

### Overview

The client component initiates tunnel connections to the server, manages encryption, and handles local traffic routing.

![Client Setup](https://via.placeholder.com/600x400?text=Client+Configuration+Setup)

### Quick Start

**Location**: `client/setup-client.rsc`

```bash
# 1. Connect to your Client MikroTik Router via SSH or Winbox
ssh admin@<client-ip>

# 2. Upload and execute the client setup script
/system script run setup-client

# 3. Verify the connection
/interface print
```

### Configuration Files

#### Main Client Script: `client/setup-client.rsc`

Establishes tunnel connections and basic routing:
- Creates VPN client connection
- Sets up tunnel parameters
- Configures default routes

```routeros
# Example snippet - Full script available in client/setup-client.rsc

# Create OVPN client connection
/interface/ovpn-client/add name=ovpn-out \
    connect-to=<server-ip> port=1194 \
    certificate=client-cert mode=ip protocol=tcp

# Enable the connection
/interface/ovpn-client/enable ovpn-out
```

#### Tunnel Configuration: `client/config/tunnel-config.rsc`

Manages tunnel parameters and encryption:
- Protocol selection (OpenVPN, IPSec, WireGuard, etc.)
- Encryption algorithm setup
- Authentication credentials
- Keep-alive settings

```routeros
# Configure tunnel encryption
/interface/ipsec-client/add name=ipsec_tunnel \
    address=<server-ip> \
    proposal=default \
    auth-method=eap
```

#### Routing Rules: `client/config/routing-rules.rsc`

Defines traffic routes through the tunnel:
- Static routes to server network
- Default gateway configuration
- Policy-based routing

```routeros
# Add route to server network via tunnel
/ip/route/add dst-address=<server-network>/24 \
    gateway=<tunnel-gateway> \
    check-gateway=ping
```

#### Security Configuration: `client/config/security-config.rsc`

Implements security best practices:
- Firewall rules for client protection
- DNS security settings
- Connection validation

```routeros
# Enable firewall
/ip/firewall/filter/add chain=forward \
    connection-state=established,related action=accept

# Configure DNS
/ip/dns/set servers=<server-dns>
```

### Client Setup Instructions

1. **Access Your Router**
   ```bash
   ssh admin@<your-client-ip>
   ```

2. **Load Configuration Scripts**
   ```bash
   /import file-name=client/setup-client.rsc
   ```

3. **Verify Installation**
   - Check tunnel status: `/interface print`
   - Verify connected state: `/interface/ovpn-client print`
   - Test routes: `/ip/route print`

4. **Test Connectivity**
   ```bash
   # Ping server through tunnel
   /ping <server-gateway-ip>
   
   # Check traffic flow
   /interface monitor-traffic ovpn-out
   ```

---

## 🚀 Usage

### Starting a Tunnel

**On Server:**
```routeros
/interface/ovpn-server/server/set enabled=yes
```

**On Client:**
```routeros
/interface/ovpn-client/enable ovpn-out
```

### Monitoring Connection Status

```routeros
# View tunnel interfaces
/interface print

# Monitor traffic
/interface monitor-traffic <interface-name>

# Check connection statistics
/ip/firewall/connection print
```

### Adjusting Bandwidth

```routeros
# Set queue for traffic limiting
/queue/simple/add name=tunnel_limit \
    target=<tunnel-interface> \
    max-limit=10M/10M
```

### Logging and Debugging

```routeros
# View recent logs
/log print tail=50

# Enable detailed logging
/system logging/set default-limit=100000 default-server-file-size=1000KiB

# Filter logs by topic
/log print topics=ipsec
```

---

## 🔧 Troubleshooting

### Common Issues

#### 1. Tunnel Not Connecting

**Symptoms**: Connection state shows "connecting" but never establishes

**Solutions**:
- Verify server is accessible: `/ping <server-ip>`
- Check firewall rules: `/ip/firewall/filter print`
- Review logs: `/log print topics=ipsec`
- Confirm credentials and certificates

#### 2. Poor Connection Speed

**Symptoms**: Tunnel established but slow throughput

**Solutions**:
- Check MTU settings: `/interface print`
- Monitor CPU usage: `/system resource print`
- Verify no queue limits: `/queue/simple print`
- Test link speed: `/interface monitor-traffic ether1`

#### 3. Intermittent Disconnections

**Symptoms**: Tunnel drops and reconnects frequently

**Solutions**:
- Increase keep-alive interval
- Check for packet loss: `/ping -c 100 <gateway>`
- Review timeout settings
- Monitor router stability: `/log print topics=interface`

#### 4. DNS Resolution Issues

**Symptoms**: Cannot resolve domain names through tunnel

**Solutions**:
- Verify DNS settings: `/ip/dns print`
- Check DNS servers are reachable
- Flush DNS cache: `/ip/dns/cache/flush`
- Use static host entries if needed

### Debug Checklist

- [ ] Network connectivity to server
- [ ] Firewall rules allow tunnel traffic
- [ ] IP addressing is correct
- [ ] Routing rules are in place
- [ ] Authentication credentials are valid
- [ ] Certificates are not expired
- [ ] Router has sufficient resources
- [ ] Logs show no errors

---

## 📚 Additional Resources

### Documentation

- [MikroTik Official Documentation](https://wiki.mikrotik.com)
- [Protocol Reference Guide](docs/protocol-reference.md)
- [Configuration Guide](docs/configuration-guide.md)

### Examples

- [Basic Tunnel Setup](examples/basic-tunnel-setup.md)
- [Advanced Scenarios](examples/advanced-scenarios.md)
- [Troubleshooting Guide](examples/troubleshooting-guide.md)

---

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

1. **Fork the Repository**
2. **Create a Feature Branch**: `git checkout -b feature/your-feature-name`
3. **Commit Changes**: `git commit -m "Add your descriptive message"`
4. **Push to Branch**: `git push origin feature/your-feature-name`
5. **Open a Pull Request**: Describe your changes and improvements

### Code Style Guidelines

- Use consistent indentation (4 spaces for scripts)
- Add comments for complex configurations
- Test all scripts before submitting
- Update documentation accordingly

---

## 📄 License

This project is licensed under the MIT License. See the LICENSE file for details.

---

## ⚠️ Disclaimer

These scripts and configurations are provided as-is for educational and practical purposes. Before deploying in production:

- Test thoroughly in a lab environment
- Backup your router configuration
- Understand all security implications
- Monitor the tunnel after deployment
- Keep MikroTik RouterOS updated

---

## 📞 Support

For issues, questions, or suggestions:

1. Check the [Troubleshooting](#troubleshooting) section
2. Review existing documentation
3. Search GitHub issues
4. Create a new issue with detailed information

---

**Last Updated**: 2026-01-09

**Maintained by**: FuncSmile
