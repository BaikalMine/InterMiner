# InterMiner

InterMiner is a GPU miner with independent mining profiles for:

- `cryptixhash`: CryptixHash v2 / OX8.
- `zelhash`: CS Coin, Equihash 125,4. `cscoin` is accepted as an alias.
- `pearlhash`: PearlHash PoUW GEMM.
- `sha3t`: BitcoinIII / BC3 triple SHA3-256.

Select the algorithm explicitly with `-a` or `--algorithm`. The default is
`cryptixhash`.

Binary releases are published in
[BaikalMine/InterMiner](https://github.com/BaikalMine/InterMiner/releases).
Source code is maintained in
[BaikalMine-Pools/b-miner](https://github.com/BaikalMine-Pools/b-miner).

## Download

Current pre-release:
[InterMiner v1.2.8](https://github.com/BaikalMine/InterMiner/releases/tag/v1.2.8)

| Platform | Asset |
| --- | --- |
| Windows x64 | [InterMiner-v1.2.8-win64-amd64.zip](https://github.com/BaikalMine/InterMiner/releases/download/v1.2.8/InterMiner-v1.2.8-win64-amd64.zip) |
| Linux x86-64 | [InterMiner-v1.2.8-linux-amd64.tar.gz](https://github.com/BaikalMine/InterMiner/releases/download/v1.2.8/InterMiner-v1.2.8-linux-amd64.tar.gz) |
| HiveOS | [InterMiner-v1.2.8-hiveos.tar.gz](https://github.com/BaikalMine/InterMiner/releases/download/v1.2.8/InterMiner-v1.2.8-hiveos.tar.gz) |

The v1.2.8 packages use the CUDA 12.8 universal build.

## Quick Start

Replace the wallet and worker placeholders before starting the miner. The pool
password defaults to `x`.

### CryptixHash

```bat
InterMiner-cuda.exe -a cryptixhash ^
  -s stratum+tcp://cytx.baikalmine.com:9010 ^
  -w YOUR_WALLET.YOUR_WORKER ^
  --gpu 0,1 --threads 0 --cuda-no-blocking-sync
```

For AMD/OpenCL:

```bat
InterMiner.exe -a cryptixhash --cuda-disable --opencl-enable ^
  -s stratum+tcp://cytx.baikalmine.com:9010 ^
  -w YOUR_WALLET.YOUR_WORKER ^
  --gpu 0,1 --threads 0
```

### CS Coin ZelHash

```bat
InterMiner.exe -a zelhash ^
  -s stratum+tcp://cs.baikalmine.com:2540 ^
  -w YOUR_WALLET.YOUR_WORKER ^
  --gpu 0,1 --threads 0
```

ZelHash selects the native CUDA backend for NVIDIA and OpenCL for AMD. Add
`--cs-opoi` only when the CS Coin pool-dispatched OPoI path is required.

### PearlHash

BaikalMine:

```bat
InterMiner-cuda.exe -a pearlhash ^
  -s stratum+tcp://pearl-ru2.baikalmine.com:2010 ^
  -w YOUR_WALLET.YOUR_WORKER ^
  --password x --gpu 0
```

HeroMiners example:

```bat
InterMiner-cuda.exe -a pearlhash ^
  -s stratum+tcp://ru.pearl.herominers.com:1200 ^
  -w YOUR_WALLET.YOUR_WORKER ^
  --password x --gpu 0
```

### SHA3T / BitcoinIII

```bat
InterMiner-cuda.exe -a sha3t ^
  -s stratum+tcp://bc3.baikalmine.com:2550 ^
  -w YOUR_WALLET.YOUR_WORKER ^
  --password x --gpu 0
```

Ready-to-edit BAT files for all profiles are included in the Windows archive.

## Algorithms

| Algorithm | Description | Backend and notes |
| --- | --- | --- |
| `cryptixhash` | CryptixHash v2 / OX8 | CUDA and OpenCL |
| `zelhash` | CS Coin, Equihash 125,4 | Native CUDA for NVIDIA, OpenCL for AMD; optional `--cs-opoi` |
| `pearlhash` | PearlHash PoUW GEMM | Native CUDA |
| `sha3t` | BitcoinIII / BC3 triple SHA3-256 | CUDA/OpenCL worker profile |

Ordinary mining does not download models or start an inference runtime. CS Coin
uses its own isolated OPoI path only when `--cs-opoi` is supplied with
`-a zelhash`.

PearlHash packages contain architecture-specific CUDA paths for RTX 20, RTX 30,
RTX 40, and RTX 50 GPUs. The Ampere/RTX 30 path has been validated on a physical
RTX 3090. Other packaged architecture paths still require validation on their
corresponding physical cards.

## Command Reference

### Pool and wallet

| Command | Description |
| --- | --- |
| `-a`, `--algorithm` | `cryptixhash`, `zelhash`, `pearlhash`, or `sha3t` |
| `-s`, `--stratum` | Pool URL; `stratum+tcp://` is optional |
| `-w`, `--wallet` | Wallet address, optionally followed by `.WORKER` |
| `--password PASSWORD` | Pool password; defaults to `x` |
| `-t`, `--threads` | CPU mining threads; defaults to `0` for GPU-only mining |
| `--cs-opoi` | Enable CS Coin OPoI; valid only with `zelhash` |
| `--debug` | Enable detailed logging |
| `--help` | Print all available commands |
| `--version` | Print the miner version |

### GPU selection and tuning

| Command | Description |
| --- | --- |
| `-g`, `--gpu 0,1` | Mine only on the selected GPU indices |
| `--devices 0,1` | Alias for `--gpu` |
| `--list-gpus` | List NVIDIA GPU indices and exit |
| `--cuda-device 0,1` | CUDA-specific device selector |
| `--opencl-device 0,1` | OpenCL-specific device selector |
| `--opencl-platform N` | Select an OpenCL platform |
| `--cuda-no-blocking-sync` | Low-latency CUDA polling mode |
| `--cuda-spin-sync` | Lowest-latency CUDA mode with higher CPU usage |
| `--cuda-workload VALUE` | Set a manual CUDA workload for testing |
| `--autotune-cache FILE` | Store CUDA auto-tune profiles in a custom file |
| `--no-autotune-cache` | Do not load or save auto-tune results for this run |
| `--reset-autotune-cache` | Clear saved CUDA auto-tune profiles before starting |

CUDA workloads are tuned per physical GPU and algorithm. Use manual workload
settings only when testing a known stable configuration. On HiveOS, configure
clocks, memory offsets, power limits, and fans through the standard rig controls.

## Monitoring

```text
--api-bind 127.0.0.1:4098
```

This enables the read-only local API:

```text
GET http://127.0.0.1:4098/api/v1/summary
GET http://127.0.0.1:4098/api/v1/health
```

The API accepts loopback addresses only. HiveOS uses it for fresh per-GPU
hashrates, uptime, and accepted/rejected counters, with log parsing as a
fallback.

## Windows

1. Download and extract `InterMiner-v1.2.8-win64-amd64.zip`.
2. Edit the appropriate included `start-*.bat` file.
3. Set the wallet, worker name, pool, and GPU list.
4. Run the script.

Use `InterMiner-cuda.exe` for NVIDIA CUDA mining. Use `InterMiner.exe` with
`--cuda-disable --opencl-enable` for OpenCL mining.

## Linux

Requirements:

- x86-64 Linux.
- A compatible NVIDIA driver for CUDA mining.
- The CUDA Toolkit math libraries only when optional CS Coin OPoI is enabled.

```bash
tar -xzf InterMiner-v1.2.8-linux-amd64.tar.gz
cd InterMiner-v1.2.8-linux-amd64
chmod +x InterMiner-cuda

LD_LIBRARY_PATH="$PWD:${LD_LIBRARY_PATH}" ./InterMiner-cuda \
  -a pearlhash \
  -s stratum+tcp://pearl-ru2.baikalmine.com:2010 \
  -w YOUR_WALLET.YOUR_WORKER \
  --password x --gpu 0
```

## HiveOS

Use this Custom Miner name:

```text
InterMiner-v1.2.8
```

Install URL:

```text
https://github.com/BaikalMine/InterMiner/releases/download/v1.2.8/InterMiner-v1.2.8-hiveos.tar.gz
```

Supported `Hash Algorithm` values:

```text
cryptixhash
zelhash
pearlhash
sha3t
```

The aliases `cryptix` and `cscoin` are normalized to `cryptixhash` and
`zelhash`. A manually supplied `--algorithm` or `-a` in the user config takes
priority over the HiveOS Hash Algorithm field.

### PearlHash Flight Sheet JSON

The following text can be used as the PearlHash Custom Miner flight-sheet
configuration. Its wallet ID must exist in the target HiveOS account.

```json
{"name":"InterMiner","isFavorite":false,"items":[{"coin":"PRL","pool_ssl":false,"wal_id":11120435,"dpool_ssl":false,"miner":"custom","miner_alt":"InterMiner-v1.2.8","miner_config":{"url":"pearl-ru2.baikalmine.com:2010","miner":"InterMiner-v1.2.8","template":"%WAL%.%WORKER_NAME%","install_url":"https://github.com/BaikalMine/InterMiner/releases/download/v1.2.8/InterMiner-v1.2.8-hiveos.tar.gz","user_config":"-a pearlhash"},"pool_geo":[]}]}
```

## Developer Fee

| Algorithm | BaikalMine pools | Other pools |
| --- | ---: | ---: |
| CryptixHash | 0.75% | 1.0% |
| CS Coin ZelHash | 1.0% | 2.0% |
| PearlHash | 0.5% | 1.0% |
| SHA3T | 1.0% | 2.0% |

PearlHash and SHA3T fee work is scheduled from accepted user shares. At 1%,
one fee share is scheduled per 100 accepted user shares. A rejected fee share
does not clear the outstanding fee work.

## Notes

- `Share accepted` confirms that the pool accepted a submitted share.
- The console shows the current hashrate and a 60-second average.
- ZelHash is reported in `Sol/s`; HiveOS receives the same per-GPU unit.
- Signed manifest verification and a content-addressed package cache are present
  as a foundation for future library updates. No remote package source is
  enabled by default.
