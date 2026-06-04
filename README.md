# Homelab

My personal homelab running on bare-metal Proxmox with dual GPUs, multiple VMs, and a full stack of self-hosted services. Built as both a learning environment and the infrastructure backbone for my AI assistant project, [Casper](https://github.com/kosmoas/casper).

---

## Hardware

| Component | Spec |
|---|---|
| Motherboard | Gigabyte Z370P-D3 |
| Storage | Samsung 970 EVO 1TB NVMe |
| GPU 0 (inference) | NVIDIA RTX 3060 12GB |
| GPU 1 | NVIDIA GTX 1650 Super 4GB |
| Hypervisor | Proxmox VE (bare metal) |

---

## Network

```
ISP
 └── Router
      ├── Proxmox Host
      │    ├── VM100 — Ubuntu ← main services
      │    ├── VM101 — Windows Server (stopped)
      │    └── VM102 — Windows 11 (stopped)
      └── All devices → Pi-hole DNS → Tailscale overlay
```

**DNS:** Pi-hole handles network-wide ad blocking and local DNS. Tailscale conflict resolved by disabling Tailscale DNS management and pointing all devices to Pi-hole.

**Remote access:** Tailscale VPN — every device on the tailnet resolves through Pi-hole and can reach homelab services securely from anywhere.

---

## Virtual Machines

| VM | OS | Purpose |
|---|---|---|
| VM100 | Ubuntu 22.04 | Primary — all Docker services, Casper AI, GPU workloads |
| VM101 | Windows Server | Lab / stopped |
| VM102 | Windows 11 | Lab / stopped |

GPU passthrough configured on VM100 — RTX 3060 assigned for LLM inference via Ollama.

---

## Services (Docker on VM100)

| Service | Port | What It Does |
|---|---|---|
| [Casper / Open WebUI](https://github.com/kosmoas/casper) | 8080 | Local AI chat interface |
| Ollama | 11434 | LLM inference engine (GPU-accelerated) |
| n8n | 5678 | Workflow automation |
| Pi-hole | 80 | Network-wide DNS + ad blocking |
| Portainer | 9000 | Docker container management UI |
| SearXNG | 8888 | Self-hosted search engine |
| Uptime Kuma | 3001 | Service uptime monitoring |
| Traccar | 8082 | GPS tracking server |
| ChromaDB | 6000 | Vector database for AI memory |
| Casper PWA | 3500 | Mobile web app for Casper |

---

## Key Configs & Decisions

**Proxmox over VMware** — Migrated from VMware after GCC 15 kernel module incompatibility made it unworkable on this hardware. Proxmox runs clean on bare metal with no driver issues.

**GPU passthrough** — RTX 3060 passed through to VM100 for all Ollama inference. BIOS quirk on this board requires the monitor plugged into the top GPU slot for GRUB to display correctly.

**Pi-hole + Tailscale DNS coexistence** — By default these conflict. Fixed by running `tailscale set --accept-dns=false` and setting Pi-hole as the global nameserver in the Tailscale admin console. Each device now shows its individual Tailscale IP in Pi-hole query logs.

**Ollama persistence** — After VM restarts, `OLLAMA_NUM_GPU=999` must be set to force GPU allocation. Handled in the startup script.

---

## Monitoring & Self-Healing

- **Uptime Kuma** — visual uptime dashboard for all services
- **Casper autonomous monitor** — Python cron running every 2 minutes, checks service health via Docker socket and Proxmox REST API, restarts dead services, sends Telegram alerts
- **Health cron** — separate 5-minute cron logs resource usage (CPU, RAM, GPU, disk) and alerts on thresholds

---

## Tech Stack

**Hypervisor:** Proxmox VE  
**OS:** Ubuntu 22.04  
**Containers:** Docker, managed via Portainer  
**Networking:** Tailscale VPN, Pi-hole DNS  
**Automation:** Bash, Python, Cron, Systemd  
**Monitoring:** Uptime Kuma, custom Python health scripts  
**AI inference:** Ollama, NVIDIA CUDA, GPU passthrough  
