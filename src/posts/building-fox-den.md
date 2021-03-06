---
title: Building the Fox Den
date: '2020-06-06'
tags:
  - homelab
  - blog
  - new
---

### Hello world!

Like everyone learning a new programming language for the first time, this blog wouldn't be complete without a `Hello World` post. Hopefully some of you will have browsed this site somewhat to get a feel of what's going on, but if not - Welcome! A background of the site and myself can be found [here](/pages/about-me), so I won't dive too deeply into that. For now, let's get to business!

---

### The Fox Den

The Fox Den is the latest iteration of many previously failed homelab attempts. This time, complete with 'new' hardware, a plan and a plethora of enterprise-grade tools behind me (Thank you Red Hat!).

Unfortunately, I've already performed most of the setup of the lab, so I won't be listing everything in detail in this post, I'll break those out into separate posts and link them here as and when they're done, but I thought I'd give you a tour of what hardware and services I'm rocking at the moment!

---

## Services
##### Write-ups coming soon
- ~~Knot DNS~~ / ~~Unbound DNS~~ / Knot DNS
- ~~Knot-Resolver~~ / Knot Resolver
- ISC DHCP Server
- UniFi Controller
- Libvirt
- Ansible Automation Platform (Soon)
- Red Hat IDM (Soon)
- Rancher k3s (Soon)

---


### Roke

```
 |       Info | Purpose: Bare-metal Hypervisor                   |
 | ---------: | ------------------------------------------------ |
 |     Model: | HP DL360 G7                                      |
 |        OS: | Red Hat Enterprise Linux 8.2                     |
 |       CPU: | 2x Intel Xeon x5650 @ 2.6GHz                     |
 |    Memory: | 24GB ECC DDR3 @ 1333MHz (6x4GB)                  |
 |   Storage: | 2x DG146A4960 (146GB - 10K - SAS)                |
 |       NIC: | 2x HP NC382i Dual Port Gigabit (4 Ports Total)   |
 | ---------- | ------------------------------------------------ |
```

### Archmage
```
 |       Info | Purpose: Bare-metal Hypervisor                   |
 | ---------: | ------------------------------------------------ |
 |     Model: | HP DL360 G7                                      |
 |        OS: | Red Hat Enterprise Linux 8.2                     |
 |       CPU: | 2x Intel Xeon x5650 @ 2.6GHz                     |
 |    Memory: | 24GB ECC DDR3 @ 1333MHz (6x4GB)                  |
 |   Storage: | 2x DG146A4960 (146GB - 10K - SAS)                |
 |       NIC: | 2x HP NC382i Dual Port Gigabit (4 Ports Total)   |
 | ---------- | ------------------------------------------------ |
```

### Gont (Still to arrive)
```
 |         Info | Purpose: Desktop Workstation                     |
 | -----------: | ------------------------------------------------ |
 |       Model: | Custom-Built.     		                       |
 |          OS: | Windows 10 20.04                                 |
 |         CPU: | AMD Ryzen 9 3900                                 |
 |      Memory: | 16GB Corsair Vengeance DDR4 3200 White RGB       |
 |     Storage: | 1TB Sabrent Rocket M.2 PCI-E Gen 4               |
 |            - | 256GB Sabrent Rocker M.2 PCI-E Gen 3             |
 |            - | 128GB Kingston HyperX Sata 3 6GBPS SSD           |
 |            - | 1TB Samsung Spinmaster 5600RPM Sata 3 HDD.       |
 |  Motherboard | Asus ROG Strix X570-E Gaming                     |
 |          GPU | Gigabyte RX570 Gaming 4GB.                       |
 |          GPU | [On arrival] Asus GeForce RTX 3080 TUF Gaming    |
 | ------------ | ------------------------------------------------ |
```

### Ged

```
 |       Info | Purpose: Daily driver laptop                     |
 | ---------: | ------------------------------------------------ |
 |     Model: | Retina MacBook Pro (2015)                        |
 |        OS: | MacOS Catalina 10.15.5                           |
 |       CPU: | Intel Core i7-4770HQ @ 2.2GHz                    |
 |    Memory: | 16GB DDR3 @ 1600MHz (2x8GB)                      |
 |   Storage: | 256GB Apple SATA SSD                             |
 |       NIC: | Apple Airport Card                               |
 | ---------- | ------------------------------------------------ |
```

### Master-Namer
```
 |       Info | Purpose: DNS + DHCP Server                       |
 | ---------: | ------------------------------------------------ |
 |     Model: | Libvirt/KVM Host                                 |
 |        OS: | Red Hat Enterprise Linux 8.2                     |
 |       CPU: | 2x vCPU                                          |
 |    Memory: | 4GiB                                             |
 |   Storage: | 20GiB                                            |
 |       NIC: | Bridge-to-LAN br0 -> enp3s0f1                    |
 | ---------- | ------------------------------------------------ |
```

### Master-Chanter
```
|       Info | Purpose: Unifi Network Controller                |
| ---------: | ------------------------------------------------ |
|     Model: | Libvirt/KVM Host                                 |
|        OS: | Ubuntu Server 20.04                              |
|       CPU: | 1x vCPU                                          |
|    Memory: | 4GiB                                             |
|   Storage: | 25GiB                                            |
|       NIC: | Bridge-to-LAN br0 -> enp3s0f1                    |
| ---------- | ------------------------------------------------ |
```