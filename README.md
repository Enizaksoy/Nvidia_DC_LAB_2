Topology Overview
Architecture: Spine-Leaf (Clos) Topology

4 Spine Switches (Layer 3)
4 Leaf Switches (Layer 2/3)
9 Servers (server-0 to server-8)

<img width="1516" height="982" alt="image" src="https://github.com/user-attachments/assets/091d9db6-ed6b-4980-933c-9ae5d633d339" />
# Data Center Network - Cumulus Linux Spine-Leaf Topology

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Cumulus Linux](https://img.shields.io/badge/Cumulus-Linux%205.x-blue)](https://www.nvidia.com/en-us/networking/ethernet-switching/cumulus-linux/)

## Overview

This repository contains the complete network infrastructure configuration for a production-grade data center using NVIDIA Cumulus Linux switches in a Spine-Leaf (Clos) topology. The design implements BGP-EVPN with VXLAN overlay for Layer 2/3 connectivity across the fabric.

### Architecture

- **4 Spine Switches**: Provides Layer 3 routing backbone
- **5 Leaf Switches**: Provides server connectivity (ToR - Top of Rack)
- **9 Servers**: Connected with redundant uplinks
- **Protocol**: BGP with EVPN address family
- **Overlay**: VXLAN for Layer 2 extension
- **Underlay**: BGP unnumbered with IPv4

## Network Topology

```
                    ┌─────────────────────────────────┐
                    │         Spine Layer             │
                    │  spine-1  spine-2  cvx-1 cvx-2  │
                    └──────────┬─┬─┬─┬──┬─┬─┬─┬──────┘
                              ╱ ╱ ╱ ╱  ╱ ╱ ╱ ╱
                             ╱ ╱ ╱ ╱  ╱ ╱ ╱ ╱
                    ┌───────────────────────────────┐
                    │         Leaf Layer             │
                    │  leaf-1  leaf-2  c-1  c-2 c-3  │
                    └──────┬─────┬─────┬────┬────┬───┘
                           │     │     │    │    │
                    ┌──────────────────────────────────┐
                    │        Server Layer              │
                    │  srv-0 ... srv-8 (9 servers)     │
                    └──────────────────────────────────┘
```

## Features

✅ **High Availability**: Redundant spine layer with ECMP load balancing  
✅ **Scalability**: Easy to add new leafs and servers  
✅ **Multi-tenancy**: VXLAN-based network segmentation  
✅ **Automation Ready**: Ansible playbooks included  
✅ **Zero-Touch Provisioning**: Automated deployment scripts  
✅ **BGP Unnumbered**: Simplified IP address management  

## Repository Structure

```
.
├── README.md                    # This file
├── topology/
│   ├── network-diagram.png      # Visual topology diagram
│   ├── connections.yaml         # Physical connection matrix
│   └── ip-plan.yaml             # IP addressing scheme
├── configs/
│   ├── spine-switches/          # Spine switch configurations
│   │   ├── spine-1/
│   │   │   ├── interfaces       # /etc/network/interfaces
│   │   │   ├── frr.conf         # FRRouting configuration
│   │   │   └── ports.conf       # Port speeds and breakout
│   │   ├── spine-2/
│   │   ├── cumulusvx-1/
│   │   └── cumulusvx-2/
│   ├── leaf-switches/           # Leaf switch configurations
│   │   ├── leaf-1/
│   │   ├── leaf-2/
│   │   ├── c-1/
│   │   ├── c-2/
│   │   └── c-3/
│   └── templates/               # Jinja2 templates
│       ├── spine-bgp.j2
│       └── leaf-evpn.j2
├── ansible/
│   ├── inventory/
│   │   ├── hosts.yaml           # Ansible inventory
│   │   └── group_vars/
│   │       ├── spines.yaml
│   │       └── leafs.yaml
│   ├── playbooks/
│   │   ├── deploy-all.yaml      # Deploy full fabric
│   │   ├── deploy-spine.yaml    # Deploy spine switches
│   │   ├── deploy-leaf.yaml     # Deploy leaf switches
│   │   ├── backup-config.yaml   # Backup configurations
│   │   └── validate-fabric.yaml # Validation checks
│   └── roles/
│       ├── common/
│       ├── spine/
│       └── leaf/
├── scripts/
│   ├── backup-configs.sh        # Backup all switch configs
│   ├── validate-bgp.py          # BGP validation script
│   ├── validate-evpn.py         # EVPN validation script
│   └── generate-configs.py      # Config generator from templates
├── docs/
│   ├── installation.md          # Installation guide
│   ├── troubleshooting.md       # Troubleshooting guide
│   ├── upgrade-procedures.md    # Upgrade procedures
│   ├── bgp-design.md            # BGP design decisions
│   └── vxlan-design.md          # VXLAN/EVPN design
└── tests/
    ├── test_connectivity.py     # Connectivity tests
    └── test_redundancy.py       # Redundancy tests
```

## Prerequisites

- **Cumulus Linux**: Version 5.0 or higher
- **Ansible**: Version 2.10+ (for automation)
- **Python**: 3.8+ (for scripts)
- **SSH Access**: To all switches
- **Management Network**: Out-of-band management configured

### Required Python Packages

```bash
pip install -r requirements.txt
```

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/datacenter-network.git
cd datacenter-network
```

### 2. Configure Ansible Inventory

Edit `ansible/inventory/hosts.yaml` with your switch management IPs:

```yaml
all:
  children:
    spines:
      hosts:
        spine-1:
          ansible_host: 192.168.1.10
        spine-2:
          ansible_host: 192.168.1.11
```

### 3. Deploy Configuration

```bash
# Deploy to all switches
ansible-playbook -i ansible/inventory/hosts.yaml ansible/playbooks/deploy-all.yaml

# Or deploy individually
ansible-playbook -i ansible/inventory/hosts.yaml ansible/playbooks/deploy-spine.yaml
ansible-playbook -i ansible/inventory/hosts.yaml ansible/playbooks/deploy-leaf.yaml
```

### 4. Validate Deployment

```bash
# Run validation playbook
ansible-playbook -i ansible/inventory/hosts.yaml ansible/playbooks/validate-fabric.yaml

# Or use validation scripts
python scripts/validate-bgp.py
python scripts/validate-evpn.py
```

## Network Design

### IP Addressing Scheme

| Component | IP Range | Purpose |
|-----------|----------|---------|
| Loopbacks (Spine) | 10.255.255.1-4/32 | BGP router-id |
| Loopbacks (Leaf) | 10.255.255.11-15/32 | BGP router-id |
| VTEP Addresses | 10.0.1.1-5/32 | VXLAN tunnel endpoints |
| Point-to-Point Links | 10.1.0.0/16 | Underlay connectivity |
| Management | 192.168.1.0/24 | Out-of-band management |

### BGP AS Numbers

- **Spine Layer**: AS 65001-65004
- **Leaf Layer**: AS 65101-65105

### VLAN to VNI Mapping

| VLAN | VNI | Description |
|------|-----|-------------|
| 10 | 10010 | Production Servers |
| 20 | 10020 | Storage Network |
| 30 | 10030 | Development |
| 99 | 10099 | Management VLAN |

## Configuration Examples

### Spine Switch BGP Configuration

```bash
router bgp 65001
 bgp router-id 10.255.255.1
 bgp bestpath as-path multipath-relax
 neighbor LEAFS peer-group
 neighbor LEAFS remote-as external
 neighbor swp1 interface peer-group LEAFS
 !
 address-family ipv4 unicast
  redistribute connected
 exit-address-family
 !
 address-family l2vpn evpn
  neighbor LEAFS activate
  advertise-all-vni
 exit-address-family
```

### Leaf Switch EVPN Configuration

```bash
router bgp 65101
 bgp router-id 10.255.255.11
 neighbor SPINES peer-group
 neighbor SPINES remote-as external
 neighbor swp1 interface peer-group SPINES
 !
 address-family l2vpn evpn
  neighbor SPINES activate
  advertise-all-vni
  advertise-svi-ip
 exit-address-family
```

## Verification Commands

### Check BGP Status

```bash
# On any switch
net show bgp summary
net show bgp l2vpn evpn summary
```

### Check EVPN VNIs

```bash
net show evpn vni
net show evpn vni detail
```

### Check VXLAN Tunnels

```bash
net show interface vxlan
bridge fdb show | grep 00:00:00:00:00:00
```

### Check MLAG Status (Leaf Switches)

```bash
net show clag
net show clag status
```

## Maintenance

### Backup Configurations

```bash
# Automated backup
./scripts/backup-configs.sh

# Manual backup via Ansible
ansible-playbook -i ansible/inventory/hosts.yaml ansible/playbooks/backup-config.yaml
```

### Upgrade Procedure

Refer to [docs/upgrade-procedures.md](docs/upgrade-procedures.md) for detailed upgrade steps.

## Troubleshooting

Common issues and solutions are documented in [docs/troubleshooting.md](docs/troubleshooting.md).

### Quick Diagnostics

```bash
# Check all BGP sessions
for switch in spine-1 spine-2 leaf-1 leaf-2; do
  ssh $switch "net show bgp summary"
done

# Validate EVPN routes
python scripts/validate-evpn.py --all
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -am 'Add new feature'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Create a Pull Request

## Testing

```bash
# Run all tests
pytest tests/

# Run specific test
pytest tests/test_connectivity.py -v
```

## Security Considerations

- All passwords are stored in Ansible Vault
- Management access is restricted to out-of-band network
- SSH keys are used for authentication
- TACACS+ integration for centralized authentication (optional)

## Performance Tuning

- MTU set to 9216 (jumbo frames enabled)
- BGP timers optimized for fast convergence
- ECMP with 64-way load balancing
- Hardware offloading enabled for VXLAN

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Authors

- Your Name - Initial work - [@yourusername](https://github.com/yourusername)

## Acknowledgments

- NVIDIA Cumulus Linux Documentation
- Data Center Network Design Best Practices
- BGP EVPN VXLAN RFC 7432

## Support

For issues and questions:
- Open an issue in this repository
- Contact: your.email@example.com
- Documentation: [Wiki](https://github.com/yourusername/datacenter-network/wiki)

## Roadmap

- [ ] Add IPv6 support
- [ ] Implement QoS policies
- [ ] Add monitoring integration (Prometheus/Grafana)
- [ ] Implement network automation with CI/CD
- [ ] Add disaster recovery procedures
- [ ] Integration with network management tools

---

**Last Updated**: October 2025  
**Status**: Production Ready ✅
