# CloudJinix BDIX Tools — Free Bangladesh Hosting Utilities

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20POSIX-orange.svg)](#requirements)
[![Peering](https://img.shields.io/badge/Peering-BDIX%20Ultra--Low%20Latency-green.svg)](https://jinix.cloud/tools/bdix-ping-test.html)
[![Maintained by](https://img.shields.io/badge/Maintained%20by-CloudJinix%20Engineering-cyan.svg)](https://jinix.cloud)

A production-grade collection of diagnostic tools, latency probes, ISP routing analyzers, and hardware benchmark utilities tailored specifically for developers, system administrators, and DevOps engineers operating servers and web services within Bangladesh.

Engineered to benchmark domestic **Bangladesh Internet Exchange (BDIX)** routing, detect international transit degradation, verify local packet latency, and validate KVM VPS hardware performance.

---

## 📑 Table of Contents

- [Why BDIX Tooling Matters](#-why-bdix-tooling-matters)
- [Included Utilities](#-included-utilities)
- [Quick Installation](#-quick-installation)
- [Tool 1: `bdix-ping-test.sh` (Interactive Latency & Peering Probe)](#tool-1-bdix-ping-testsh)
- [Tool 2: `bd-vps-bench.sh` (Storage & Network Benchmark)](#tool-2-bd-vps-benchsh)
- [Tool 3: `isp-route-audit.sh` (BDIX vs Transit Routing Audit)](#tool-3-isp-route-auditsh)
- [Benchmark Results: CloudJinix vs Global VPS](#-benchmark-comparison)
- [Automated Integration (Cron & Monitoring)](#-automated-integration)
- [Cloud Hosting in Bangladesh](#-cloud-hosting-in-bangladesh)
- [Contributing](#-contributing)
- [License & Attribution](#-license--attribution)

---

## 🇧🇩 Why BDIX Tooling Matters

Traditional benchmarking suites like YABS (`yet-another-bench-script`) or Geekbench test network throughput against European, North American, or Singaporean hyper-scaler data centers. When deploying customer-facing applications in Bangladesh (such as eCommerce stores, fintech APIs, ERP systems, or news portals):

1. **Domestic Latency Trumps International Bandwidth:** A 10Gbps link routing through Singapore or Mumbai introduces 45ms to 95ms of round-trip latency. A true BDIX-peered server delivers **1.2ms to 5ms** latency nationwide.
2. **Asymmetric ISP Peering:** Some Bangladeshi local ISPs route traffic over local IXPs (BDIX, NIX) while failing over to expensive upstream transit (Tata, Bharti Airtel, Cogent) during peak hours, causing unexplained latency spikes.
3. **Resource Overselling:** Budget VPS providers frequently oversell CPU and disk IOPS on legacy OpenVZ containers. Real KVM virtualization with dedicated NVMe storage is critical for consistent production uptime.

This suite provides empirical testing tools to audit your network path and hardware integrity directly from the Linux command line.

---

## 🛠 Included Utilities

| Script Name | Purpose | Target Execution | Dependencies |
| :--- | :--- | :--- | :--- |
| `bdix-ping-test.sh` | Probes ICMP latency across major BD IXPs, ISPs, and CDNs | Local Workstation or Remote VPS | `curl`, `ping`, `awk` |
| `bd-vps-bench.sh` | Evaluates CPU sysbench, NVMe fio disk IOPS, and domestic vs global throughput | Linux VPS (Ubuntu/Debian/Alma) | `fio`, `sysbench`, `curl` |
| `isp-route-audit.sh` | Detects ASN, public IP BDIX peering status, and hop-by-hop route health | Any Linux Terminal | `traceroute` or `mtr`, `jq` |
| `nginx-bdix-tune.sh` | Pre-configured Nginx template optimized for low-latency domestic delivery | Web Server | `nginx` |

---

## 🚀 Quick Installation

You can run any utility directly via `curl`, or clone this repository to inspect and execute locally:

```bash
# Clone the repository
git clone https://github.com/cloudjinix/cloudjinix-bdix-tools.git
cd cloudjinix-bdix-tools

# Make all scripts executable
chmod +x bin/*.sh
```

Or execute directly in memory without cloning:

```bash
# Run the interactive BDIX Latency Probe
curl -fsSL https://raw.githubusercontent.com/cloudjinix/cloudjinix-bdix-tools/main/bin/bdix-ping-test.sh | bash
```

> **Web Version:** Prefer a browser GUI? Use the live online tool at **[CloudJinix BDIX Ping Test](https://jinix.cloud/tools/bdix-ping-test.html)**.

---

## Tool 1: `bdix-ping-test.sh`

Measures real-time round-trip latency (RTT) against key peering nodes across Dhaka, Chittagong, and major Bangladesh internet backbones (AmberIT, Link3, Carnival, Dot Internet, Grameenphone, Robi, and BDIX peering members).

### Usage:
```bash
./bin/bdix-ping-test.sh [options]

Options:
  -c, --count <num>    Number of ICMP pings per target (Default: 4)
  -j, --json           Output results in JSON format for automated monitoring
  -v, --verbose        Show raw ping output
  -h, --help           Display help menu
```

### Sample Output:
```text
========================================================================
    CloudJinix BDIX Latency & Peering Audit — v1.4.0
    Host Node: edge-dhk-01.jinix.cloud | IPv4: 103.x.x.x
========================================================================

 TARGET NETWORK / ROUTE        PEERING TYPE    MIN RTT   AVG RTT   PACKET LOSS
 -----------------------------------------------------------------------
 [IXP] BDIX Core Exchange      Direct Peering   0.98 ms   1.12 ms   0.0% [EXCELLENT]
 [ISP] AmberIT Dhaka           BDIX Metro       1.15 ms   1.34 ms   0.0% [EXCELLENT]
 [ISP] Link3 Technologies      BDIX Peering     1.42 ms   1.61 ms   0.0% [EXCELLENT]
 [ISP] Carnival Internet       BDIX Domestic    1.08 ms   1.25 ms   0.0% [EXCELLENT]
 [ISP] Dot Internet            BDIX Direct      1.22 ms   1.40 ms   0.0% [EXCELLENT]
 [MNO] Grameenphone 4G Core    Local Transit    14.2 ms   18.1 ms   0.0% [GOOD]
 [MNO] Robi Axiata 4G Core     Local Transit    16.5 ms   21.3 ms   0.0% [GOOD]
 [CDN] Cloudflare Dhaka (DAC)  Edge Anycast     1.02 ms   1.18 ms   0.0% [EXCELLENT]
 [REF] Singapore Hyper-scaler  International    42.1 ms   45.8 ms   0.0% [TRANSIT]
 -----------------------------------------------------------------------
 SUMMARY: 7/8 domestic targets responding under 2.5ms. BDIX route optimal.
```

---

## Tool 2: `bd-vps-bench.sh`

A comprehensive benchmarking script for Linux VPS and bare-metal nodes. Tests:
1. **CPU Performance:** Multi-threaded prime number computation via `sysbench`.
2. **Storage Latency & IOPS:** Sequential and 4K random read/write speeds using `fio` directly on your NVMe/SSD partition.
3. **Network Throughput:** Tests multi-stream download and upload speeds against BDIX nodes in Dhaka vs overseas nodes (Singapore, Frankfurt, US-East).

### Usage:
```bash
# Run complete VPS benchmark
curl -fsSL https://raw.githubusercontent.com/cloudjinix/cloudjinix-bdix-tools/main/bin/bd-vps-bench.sh | bash
```

### Script Source Breakdown:
```bash
#!/usr/bin/env bash
# CloudJinix VPS Storage & Peering Benchmark Engine
set -euo pipefail

echo "==> Auditing Disk IOPS (4k Random Read/Write with Direct I/O)..."
fio --name=randwrite --ioengine=libaio --iodepth=64 --rw=randwrite \
    --bs=4k --direct=1 --size=512M --numjobs=2 --runtime=15 \
    --group_reporting --minimal | awk -F';' '{printf "Write IOPS: %s | Bandwidth: %0.2f MB/s\n", $48, $47/1024}'

fio --name=randread --ioengine=libaio --iodepth=64 --rw=randread \
    --bs=4k --direct=1 --size=512M --numjobs=2 --runtime=15 \
    --group_reporting --minimal | awk -F';' '{printf "Read IOPS: %s | Bandwidth: %0.2f MB/s\n", $8, $7/1024}'
```

---

## Tool 3: `isp-route-audit.sh`

Identifies whether your current public IP is directly peering with BDIX or if your ISP is routing through high-latency international submarine cables (SEA-ME-WE 4 / SEA-ME-WE 5).

### Usage:
```bash
./bin/isp-route-audit.sh
```

### Diagnostic Logic:
- Fetches Origin ASN and ISP Organization from local peering databases.
- Performs multi-hop traceroute to BDIX Looking Glass endpoints.
- Alerts user if latency exceeds 15ms inside Dhaka, signaling ISP upstream misconfiguration.

---

## 📊 Benchmark Comparison

Empirical performance comparison between a standard entry-level **CloudJinix Proxmox KVM Nano VPS** and legacy offshore virtual servers accessed from Bangladesh:

| Metric / Benchmark | CloudJinix KVM VPS | Offshore VPS (Singapore) | Legacy OpenVZ Host |
| :--- | :--- | :--- | :--- |
| **Pricing** | **৳৩০০/mo (41 Poisa/hr)** | ~$6.00/mo + 15% Bank Tax | ৳800/mo - ৳1,200/mo |
| **Virtualization** | Dedicated Proxmox KVM | KVM | Oversold OpenVZ / LXC |
| **Average Dhaka Ping** | **1.2 ms** | 42.0 ms - 65.0 ms | 8.0 ms - 15.0 ms |
| **Chittagong / Sylhet Ping**| **4.5 ms** | 48.0 ms - 70.0 ms | 14.0 ms - 22.0 ms |
| **4K Random Read IOPS** | **45,000+ IOPS (Pure NVMe)**| 20,000 IOPS | 4,000 IOPS (SATA SSD) |
| **Payment Gateway** | **bKash, Nagad, Rocket (Instant)**| Dual-Currency Credit Card | Manual Bank Slip / Ticket |
| **Setup Time** | **Automated (~3 seconds)** | Automated | Manual (2 - 12 hours) |

Explore full specifications and deploy servers at **[CloudJinix High-Performance KVM VPS](https://jinix.cloud/vps)**.

---

## ⚡ Automated Integration

### Prometheus / Grafana Exporter Integration
You can pipe the JSON output of `bdix-ping-test.sh` directly into Prometheus Node Exporter via cron:

```bash
# Add to /etc/cron.d/bdix-monitor
*/5 * * * * root /usr/local/bin/bdix-ping-test.sh --json > /var/lib/node_exporter/textfile_collector/bdix.prom
```

---

## 🌐 Cloud Hosting in Bangladesh

This repository is maintained and open-sourced by **CloudJinix**, Bangladesh's developer-first cloud infrastructure platform.

If you are building web applications, deploying microservices, hosting WordPress, or running enterprise databases in Bangladesh, explore our domestic cloud ecosystem:

- **[CloudJinix Web Hosting](https://jinix.cloud):** Proprietary JinixPanel cloud hosting starting from only **৳৫৯/month**, powered by NVMe storage and LiteSpeed enterprise caching.
- **[CloudJinix KVM Cloud VPS](https://jinix.cloud/vps):** True hardware-isolated Proxmox KVM instances starting from **৳৩০০/month** with real hourly billing from **41 Poisa/hour (৳0.41/hr)** and instant bKash/Nagad checkout.
- **[Free Lifetime eCommerce (dokaan.shop)](https://jinix.cloud/ecommerce):** Zero-cost online shop builder (`username.dokaan.shop`) with automated Pathao and Steadfast courier shipping for F-commerce merchants.
- **[Live BDIX Network Speed Test](https://jinix.cloud/tools/bdix-ping-test.html):** Test your current ISP connection latency directly against Dhaka datacenter nodes.

---

## 🤝 Contributing

We welcome community contributions from network engineers, developers, and sysadmins across Bangladesh!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AddChittagongNodes`)
3. Commit your Changes (`git commit -m 'feat: add CTG IX peering endpoints'`)
4. Push to the Branch (`git push origin feature/AddChittagongNodes`)
5. Open a Pull Request

---

## 📜 License & Attribution

Distributed under the **MIT License**. See `LICENSE` for more information.

**Maintained by CloudJinix Engineering — Bangladesh BDIX Cloud Platform**  
Website: [https://jinix.cloud](https://jinix.cloud) | Documentation: [https://jinix.cloud/docs](https://jinix.cloud) | Support: 24/7 Hotline & WhatsApp
