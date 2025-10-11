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
✅ **Mul
