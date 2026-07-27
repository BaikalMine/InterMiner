# InterMiner

InterMiner is a GPU miner with two independent mining profiles:

- `keryxhash` for KeryxHash, OPoI, and PoM.
- `cryptixhash` for CryptixHash v2 (OX8).

Binary releases are published in [BaikalMine/InterMiner](https://github.com/BaikalMine/InterMiner/releases). Source code is maintained in [BaikalMine-Pools/b-miner](https://github.com/BaikalMine-Pools/b-miner).

## Download

Current release: [InterMiner v1.2.2](https://github.com/BaikalMine/InterMiner/releases/tag/v1.2.2)

| Platform | Asset |
| --- | --- |
| Windows x64 | `InterMiner-v1.2.2-win64-amd64.zip` |
| Linux x64 | `InterMiner-v1.2.2-linux-amd64.tar.gz` |
| HiveOS | `InterMiner-v1.2.2-hiveos.tar.gz` |

## Quick Start

### KeryxHash, NVIDIA CUDA

```bat
InterMiner-cuda.exe -a keryxhash ^
  -s stratum+tcp://krx.baikalmine.com:9020 ^
  -w keryx:YOUR_WALLET.YOUR_WORKER ^
  --threads 0 --cuda-no-blocking-sync
```

### KeryxHash, BaikalMine solo

```bat
InterMiner-cuda.exe -a keryxhash ^
  -s stratum+tcp://krx-solo.baikalmine.com:9021 ^
  -w keryx:YOUR_WALLET.YOUR_WORKER ^
  --threads 0 --cuda-no-blocking-sync
```

### CryptixHash, NVIDIA CUDA

```bat
InterMiner-cuda.exe -a cryptixhash ^
  -s stratum+tcp://cytx.baikalmine.com:9010 ^
  -w cryptix:YOUR_WALLET.YOUR_WORKER ^
  --threads 0 --cuda-no-blocking-sync
```

### CryptixHash, AMD OpenCL

```bat
InterMiner.exe -a cryptixhash --cuda-disable --opencl-enable ^
  -s stratum+tcp://cytx.baikalmine.com:9010 ^
  -w cryptix:YOUR_WALLET.YOUR_WORKER ^
  --threads 0
```

Replace `YOUR_WALLET` and `YOUR_WORKER` before starting the miner.

## Algorithms

| Algorithm | Description | Models, OPoI, and PoM |
| --- | --- | --- |
| `keryxhash` | KeryxHash mining profile | Enabled when required by the pool |
| `cryptixhash` | CryptixHash v2 (OX8) pure PoW profile | Disabled |

`keryxhash` is the default algorithm. CryptixHash does not start OPoI, PoM,
IPFS, Escrow, or model downloads.

For KeryxHash, InterMiner first connects to the pool and receives the current
DAA. It then selects only the model set valid for that network era. Models are
verified or downloaded after DAA is known, so obsolete models are not loaded
after a hardfork.

## Command Reference

### Pool and wallet

| Command | Description |
| --- | --- |
| `-a`, `--algorithm` | Algorithm: `keryxhash` or `cryptixhash` |
| `-s`, `--stratum` | Pool URL |
| `--keryxd-address` | Legacy alias for `--stratum` |
| `-w`, `--wallet` | Wallet address, optionally followed by `.WORKER` |
| `--mining-address` | Legacy alias for `--wallet` |
| `-t`, `--threads` | CPU mining threads; use `0` for GPU-only mining |
| `--debug` | Enable detailed logging |
| `--help` | Print all available commands |
| `--version` | Print the miner version |

Example:

```text
-a keryxhash
-s stratum+tcp://krx.baikalmine.com:9020
-w keryx:YOUR_WALLET.rig01
--threads 0
```

### GPU selection

| Command | Description |
| --- | --- |
| `-g`, `--gpu 0,1` | Mine only on the selected GPU indices |
| `--devices 0,1` | Legacy alias for `--gpu` |
| `--list-gpus` | List NVIDIA GPU indices and exit |
| `--cuda-device 0,1` | CUDA-specific device selector |
| `--opencl-device 0,1` | OpenCL-specific device selector |
| `--opencl-platform N` | Select an OpenCL platform |

Use either the portable `--gpu` selector or a backend-specific selector. Do
not combine them.

```bat
InterMiner-cuda.exe -a cryptixhash --gpu 0,2 ^
  -s stratum+tcp://cytx.baikalmine.com:9010 ^
  -w cryptix:YOUR_WALLET.worker
```

### CUDA runtime and auto-tuning

| Command | Description |
| --- | --- |
| `--cuda-no-blocking-sync` | Recommended low-latency CUDA polling mode |
| `--cuda-spin-sync` | Lowest latency mode with higher CPU usage |
| `--cuda-workload VALUE` | Manually set the CUDA workload multiplier |
| `--cuda-workload-absolute` | Treat workload values as absolute nonce counts |
| `--autotune-cache FILE` | Store the auto-tune profile in a custom file |
| `--no-autotune-cache` | Do not load or save auto-tune results for this run |
| `--reset-autotune-cache` | Clear saved CUDA auto-tune profiles before starting |

CUDA workload is tuned automatically per algorithm and GPU. Use a manual
workload only when testing a known stable configuration.

### NVIDIA clocks and power

These commands apply only to NVIDIA CUDA mining. Supply one value for every
selected GPU, or one value that applies to all selected GPUs.

| Command | Description |
| --- | --- |
| `--gpu-core-clock MHZ[,MHZ...]` | Lock NVIDIA core clocks |
| `--gpu-memory-clock MHZ[,MHZ...]` | Lock NVIDIA memory clocks |
| `--gpu-power-limit W[,W...]` | Set NVIDIA power limits |
| `--gpu-reset-tuning` | Restore default clocks and power limits |

```bat
InterMiner-cuda.exe -a cryptixhash --gpu 0,1 ^
  --gpu-core-clock 1600,1600 ^
  --gpu-memory-clock 9501,9501 ^
  --gpu-power-limit 258,258 ^
  -s stratum+tcp://cytx.baikalmine.com:9010 ^
  -w cryptix:YOUR_WALLET.worker
```

Clock and power controls require a compatible NVIDIA driver and may require
administrator privileges. On HiveOS, use the standard rig overclock settings
where possible.

### OpenCL

| Command | Description |
| --- | --- |
| `--opencl-enable` | Enable OpenCL mining |
| `--cuda-disable` | Disable CUDA workers |
| `--opencl-workload VALUE` | Set OpenCL workload |
| `--opencl-workload-absolute` | Treat workload values as absolute nonce counts |
| `--opencl-amd-disable` | Disable AMD OpenCL mining |
| `--opencl-no-amd-binary` | Do not use a precompiled AMD kernel |

For AMD mining, use `InterMiner.exe` with `--cuda-disable --opencl-enable`.

## KeryxHash Models

Model selection applies only to `keryxhash`.

| Option | VRAM class |
| --- | --- |
| `--very-light` | 6 GB |
| `--light` | 8 GB |
| Default | 12 GB |
| `--high` | 24 GB |
| `--very-high` | 32 GB |
| `--gpu-models very-high,light,default` | Set a maximum tier per CUDA GPU |

A selected tier is a maximum, not a forced upgrade. InterMiner can lower the
tier when VRAM is insufficient, but it does not select a larger tier
automatically.

Models are verified before use. Missing current models are downloaded only
after the pool sends DAA. Legacy model folders may remain on disk, but are not
used by the active KeryxHash profile.

`--cpu-inference` explicitly runs OPoI inference on the CPU. This keeps the
GPU available for hashing, but GPU inference is normally faster.

## Monitoring

```text
--api-bind 127.0.0.1:4098
```

This enables a read-only local API:

```text
GET http://127.0.0.1:4098/api/v1/summary
GET http://127.0.0.1:4098/api/v1/health
```

The API accepts loopback addresses only and cannot modify miner settings.

## Windows

1. Download and extract `InterMiner-v1.2.2-win64-amd64.zip`.
2. Edit the appropriate included `start-*.bat` file.
3. Set the wallet and worker name.
4. Run the script.

Use `InterMiner-cuda.exe` for NVIDIA CUDA mining. Use `InterMiner.exe` for
OpenCL mining.

## Linux

Requirements:

- x86_64 Linux.
- NVIDIA driver with CUDA support for CUDA mining.
- CUDA 12 runtime libraries.

```bash
tar -xzf InterMiner-v1.2.2-linux-amd64.tar.gz
cd InterMiner-v1.2.2-linux-amd64
chmod +x InterMiner-cuda

LD_LIBRARY_PATH="$PWD:${LD_LIBRARY_PATH}" ./InterMiner-cuda \
  -a keryxhash \
  -s stratum+tcp://krx.baikalmine.com:9020 \
  -w keryx:YOUR_WALLET.YOUR_WORKER \
  --threads 0 --cuda-no-blocking-sync
```

## HiveOS

Use this Custom Miner name:

```text
InterMiner-v1.2.2
```

Use this install URL:

```text
https://github.com/BaikalMine/InterMiner/releases/download/v1.2.2/InterMiner-v1.2.2-hiveos.tar.gz
```

In the HiveOS `Hash Algorithm` field, select one of:

```text
keryxhash
cryptixhash
```

HiveOS passes this selection to InterMiner as `--algorithm` and reports the
same algorithm in miner statistics. The Flight Sheet supplies the pool URL and
wallet automatically.

Recommended NVIDIA user config:

```text
--threads 0 --cuda-no-blocking-sync
```

Recommended AMD/OpenCL user config:

```text
--threads 0 --cuda-disable --opencl-enable
```

A manually supplied `--algorithm` or `-a` in user config takes priority over
the Hash Algorithm field. During upgrades, the HiveOS package searches older
InterMiner and Keryx miner directories and migrates existing `models/` data
when possible.

## BaikalMine Solo

When mining solo on BaikalMine pools, InterMiner supports automatic Escrow
reward claims.

Do not share or delete `escrow.key`. It stores the local key required to claim
these rewards.

## Developer Fee

| Algorithm | BaikalMine pools | Other pools |
| --- | --- | --- |
| KeryxHash | 1.5% | 3.0% |
| CryptixHash | 0.75% | 1.0% |

## Notes

- Models are not included in release archives.
- Use `--threads 0` for ordinary GPU-only mining.
- `Share accepted` confirms that the pool accepted a submitted share.
- The console shows the current hashrate and a 60-second average. KeryxHash
  can temporarily pause GPU hashing while GPU inference is active.
