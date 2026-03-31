# gpu-cluster-lab

> Infrastructure and tooling for large-scale GPU deployments. Accumulated from running ~300 GPU clusters (AMD + NVIDIA) at Ayla Corp (2020–2022) and ongoing freelance AI infrastructure work.

## Experience Background

- **Ayla Corp (Ukraine, 2020–2022)** — Deployed and maintained ~300 GPU (AMD + NVIDIA) infrastructure for AI compute
- **Freelance (2022–present)** — High-performance GPU setups for LLM inference and training
- Primary focus: AMD ROCm on Linux, multi-node coordination, cost-optimized inference

## Cluster Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Load Balancer                          │
│              (nginx / custom router)                     │
├──────────────┬──────────────┬──────────────┬────────────┤
│   Node 01    │   Node 02    │   Node 03    │  Node N    │
│  5x RX7900   │  5x RX7900   │  5x RX7900   │  ...       │
│  ROCm 6.x    │  ROCm 6.x    │  ROCm 6.x    │            │
├──────────────┴──────────────┴──────────────┴────────────┤
│               Shared Storage (NFS / Ceph)                │
├─────────────────────────────────────────────────────────┤
│              Monitoring (Prometheus + Grafana)           │
└─────────────────────────────────────────────────────────┘
```

## Key Scripts

```bash
# Multi-GPU health check
./scripts/gpu-health-check.sh

# Flash attention enable / VRAM optimization
./scripts/enable-flash-attn.sh

# Kernel tuning for ROCm (tested on gfx1100)
./scripts/rocm-kernel-tune.sh

# Benchmark across all nodes
./scripts/cluster-benchmark.sh --model qwen2.5-72b --nodes all
```

## ROCm Kernel Tuning

Tested configurations for maximum throughput on AMD gfx1100 (RX 7900 XTX):

```bash
# /etc/udev/rules.d/70-amdgpu.rules
echo "manual" > /sys/class/drm/card0/device/power_dpm_force_performance_level
echo "1000" > /sys/class/drm/card0/device/pp_power_profile_mode

# Memory clock tuning
rocm-smi --setmemclk 1250

# Fan curve for sustained inference
./scripts/set-fan-curve.sh --target-temp 75
```

## Flash Attention Configuration

```bash
# Max context with flash-attn on 5x GPU (80B models)
--ctx-size 131072 --flash-attn --n-gpu-layers 99 --tensor-split 0,1,2,3,4
```

## Directory Structure

```
gpu-cluster-lab/
├── scripts/
│   ├── gpu-health-check.sh
│   ├── rocm-kernel-tune.sh
│   ├── enable-flash-attn.sh
│   ├── multi-gpu-setup.sh
│   └── cluster-benchmark.sh
├── configs/
│   ├── rocm/
│   ├── prometheus/
│   └── grafana/
├── ansible/
│   └── playbooks/
├── docker/
└── docs/
    ├── rocm-installation.md
    ├── multi-node-setup.md
    └── troubleshooting.md
```

## License

MIT — Léo Camus / [NextGen Labs](https://nextgen-labs.net)
