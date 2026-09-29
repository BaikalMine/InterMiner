# InterMiner

InterMiner is a GPU miner with independent mining profiles for:

- `cryptixhash`: CryptixHash v2 / OX8.
- `zelhash`: CS Coin, Equihash 125,4. `cscoin` is accepted as an alias.
- `pearlhash`: PearlHash PoUW GEMM.
- `sha3t`: BitcoinIII / BC3 triple SHA3-256.
- `blake2b`: Bitcoin BLAKE2b / BTCB2 header-v2 profile 0. `btcb2` is an alias.
- `quantus`: Quantus / Poseidon2.
- `ycash`: Ycash (YEC), Equihash 192,7. `yec` is an alias.
- `zcl`: Zclassic (ZCL), Equihash 192,7.

Select the algorithm explicitly with `-a` or `--algorithm`. The default is
`cryptixhash`.

Binary releases are published in
[BaikalMine/InterMiner](https://github.com/BaikalMine/InterMiner/releases).
Source code is maintained in
[BaikalMine-Pools/b-miner](https://github.com/BaikalMine-Pools/b-miner).

## Download

Current pre-release:
[InterMiner v1.2.8-6](https://github.com/BaikalMine/InterMiner/releases/tag/v1.2.8-6)

| Platform | Asset |
| --- | --- |
| Windows x64 | [InterMiner-v1.2.8-6-win64-amd64.zip](https://github.com/BaikalMine/InterMiner/releases/download/v1.2.8-6/InterMiner-v1.2.8-6-win64-amd64.zip) |
| Linux x86-64 | [InterMiner-v1.2.8-6-linux-amd64.tar.gz](https://github.com/BaikalMine/InterMiner/releases/download/v1.2.8-6/InterMiner-v1.2.8-6-linux-amd64.tar.gz) |
| HiveOS | [InterMiner-v1.2.8-6-hiveos.tar.gz](https://github.com/BaikalMine/InterMiner/releases/download/v1.2.8-6/InterMiner-v1.2.8-6-hiveos.tar.gz) |
| SHA-256 checksums | [SHA256SUMS.txt](https://github.com/BaikalMine/InterMiner/releases/download/v1.2.8-6/SHA256SUMS.txt) |

The v1.2.8-6 packages use the CUDA 12.8 universal build.

## What's New in v1.2.8-6

- Added Ycash (YEC), Zclassic (ZCL), and Quantus.
- Removed OPoI support.
- Windows, Linux, and HiveOS packages are available.

YEC, ZCL, Quantus, and Bitcoin BLAKE2b require NVIDIA CUDA; AMD/OpenCL and
CPU mining are not available for these profiles. Bitcoin BLAKE2b targets
BTCB2 header-v2 profile 0, not Bitcoin II (BC2), Sia, or other protocols.

Extract the complete package into one directory. Do not mix its executable
with plugins from an older release.

### Retained PearlHash and CMP Changes

PearlHash automatically compares `compact-fused` and `direct-fused`, or
reuses a compatible cached choice, for the full materialized matrix on
supported NVIDIA Tensor Core architectures:
SM75, SM80, SM86, SM89, SM90, SM100/103, and SM120/121. This includes RTX
20/30/40/50 and supported Turing, Ampere, Hopper, and Blackwell compute GPUs.
These packages are x86-64 host builds, not ARM64 builds.

The new kernel uses a shared-memory transcript and a compact exact-integer
Tensor Core loop. CPU/GPU proof validation remains enabled. Hardware-specific
performance must be tested: RTX 3090 tests do not establish CMP 50HX, RTX
40/50, H100/H200, or B200/B300 hashrates. No universal speedup is promised.

Set the following variable in the miner process environment to compare paths:

| Value of `INTERMINER_PEARL_KERNEL` | Behavior |
| --- | --- |
| `auto` | Reuse a compatible cached choice or compare compact/direct while mining |
| `compact-fused` | Require the compact full-matrix kernel; incompatible settings fail |
| `direct-fused` | Previous non-compact fused kernel on supported architectures |
| `turing-fused` | Previous SM75 fused kernel, RTX 20 / CMP 50HX only |
| `legacy` | WMMA compatibility path on Tensor Core GPUs |

For Windows CMD, use `set INTERMINER_PEARL_KERNEL=direct-fused` before the
mining command. On Linux, prefix the command with
`INTERMINER_PEARL_KERNEL=direct-fused`. Remove the override or use `auto`
to restore automatic selection. The initial comparison takes about six minutes;
keep GPU load stable during tuning. Existing BAT files with explicit kernel
overrides keep those overrides.

Automatic selection retains the smaller-memory fallback when the full fused
matrix cannot fit. Custom matrix sizes and reduced-memory profiles retain
their existing non-compact paths. Pascal/Volta compatibility paths remain.

The earlier blocking-wait CPU-load and matrix packing fixes are retained.
Linux/HiveOS CMP auto mode skips unsupported or mixed fleets without
attempting activation. Explicit unsupported targets and activation or
verification errors still fail. This release does not expand the CMP unlock
allowlist or change clocks, power limits, fan settings, or existing algorithms'
developer fees.

CMP unlock remains Linux x86-64 only and restricted to the bundled provider's
validated profiles. Windows CMP unlock is not included. Temporary activation
can reload the NVIDIA driver; it does not change HiveOS overclock settings.

## Quick Start

Replace the wallet and worker placeholders before starting the miner. The pool
password defaults to `x`. Windows examples below use CMD/BAT syntax (`^` for
line continuation), not PowerShell. Use your own payout wallet, not a developer
fee address copied from source code.

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

ZelHash selects the native CUDA backend for NVIDIA and OpenCL for AMD.

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

### Bitcoin BLAKE2b / BTCB2

```bat
InterMiner-cuda.exe -a blake2b ^
  -s stratum+tcp://stratum.minepoolis.com:4481 ^
  -w YOUR_WALLET.YOUR_WORKER ^
  --password d=1 --gpu 0
```

The password requests lower share difficulty for GPU mining; the pool controls
the final value. `start-blake2b-minepoolis.bat` is included in the Windows archive.

### Quantus Pool

LuckyPool example:

```bat
InterMiner-cuda.exe -a quantus ^
  -s stratum+tcp://ru.lproute.com:5660 ^
  -w YOUR_QUANTUS_WALLET.YOUR_WORKER ^
  --gpu 0 --threads 0
```

Use port `5660` for this Quantus example, not the PearlHash port `3361`.
The Windows package includes `start-quantus-luckypool.bat`.
Use a LuckyPool-compatible TCP pool; other Stratum dialects and TLS pool URLs
are not interchangeable with this profile.

### Ycash (YEC)

NinjaRaider example:

```bat
InterMiner-cuda.exe -a yec ^
  -s stratum+tcp://ninjaraider.com:44560 ^
  -w YOUR_YEC_WALLET.YOUR_WORKER ^
  --gpu 0 --threads 0
```

`-a ycash` and `-a yec` select the same profile. The Windows package includes
`start-yec-ninjaraider.bat`. No separate personalization option is needed.

### Zclassic (ZCL)

zpool example, assuming ZCL payouts are supported by the pool:

```bat
InterMiner-cuda.exe -a zcl ^
  -s stratum+tcp://equihash192.eu.mine.zpool.ca:2192 ^
  -w YOUR_ZCL_WALLET.YOUR_WORKER ^
  --password c=ZCL,zap=ZCL --gpu 0 --threads 0
```

The Windows package includes `start-zcl-zpool.bat`. For YEC on the same
endpoint, use `-a yec`, a YEC payout wallet, and
`--password c=YEC,zap=YEC`, provided the pool supports YEC payouts.

On zpool, `c=` selects the payout currency and `zap=` selects the mined coin.
Match the wallet to the payout currency and confirm the pool's payout options
using the [zpool setup instructions](https://zpool.ca/).
Use one coin in `zap=`, not the literal value `YEC/ZCL`. These are setup
examples, not a claim that every payout currency or pool is always available.

### Quantus Node

For your own Quantus node, use the node's mining endpoint and authentication
files obtained over a trusted channel:

```bat
InterMiner-cuda.exe -a quantus ^
  -s quic://NODE_HOST:9833 ^
  --auth-token-file miner-auth-token ^
  --tls-cert-sha256-file miner-tls-cert-sha256 ^
  --gpu 0 --threads 0
```

Replace the host and port with your node's configured mining endpoint.
Rewards are configured on the node; `--wallet` is not required in this mode.
Do not place the authentication token in a wallet field or URL.
Direct node mining has no developer fee.

Ready-to-edit BAT files are included in the Windows archive.

## Algorithms

| Algorithm | Description | Backend and notes |
| --- | --- | --- |
| `cryptixhash` | CryptixHash v2 / OX8 | CUDA and OpenCL |
| `zelhash` | CS Coin, Equihash 125,4 | Native CUDA for NVIDIA, OpenCL for AMD |
| `pearlhash` | PearlHash PoUW GEMM | Native CUDA |
| `sha3t` | BitcoinIII / BC3 triple SHA3-256 | CUDA/OpenCL worker profile |
| `blake2b` | Bitcoin BLAKE2b / BTCB2 header-v2 profile 0 | NVIDIA CUDA only; `btcb2` alias |
| `quantus` | Quantus / Poseidon2 | NVIDIA CUDA; compatible TCP pools or authenticated QUIC nodes |
| `ycash` | Ycash (YEC), Equihash 192,7 | NVIDIA CUDA; `yec` alias |
| `zcl` | Zclassic (ZCL), Equihash 192,7 | NVIDIA CUDA |

Ordinary mining does not download models or start an inference runtime.

PearlHash packages contain architecture-specific CUDA paths for RTX 20, RTX 30,
RTX 40, and RTX 50 GPUs. The Ampere/RTX 30 path has been validated on a physical
RTX 3090. Other packaged architecture paths still require validation on their
corresponding physical cards.

## Command Reference

### Pool and wallet

| Command | Description |
| --- | --- |
| `-a`, `--algorithm` | `cryptixhash`, `zelhash`, `pearlhash`, `sha3t`, `blake2b`, `quantus`, `ycash` / `yec`, or `zcl` |
| `-s`, `--stratum` | Pool URL; `stratum+tcp://` is optional |
| `-w`, `--wallet` | Wallet address, optionally followed by `.WORKER` |
| `--password PASSWORD` | Pool password; defaults to `x` |
| `-t`, `--threads` | CPU mining threads; defaults to `0` for GPU-only mining |
| `--debug` | Enable detailed logging |
| `--help` | Print all available commands |
| `--version` | Print the miner version |

### Quantus node authentication

| Command | Description |
| --- | --- |
| `--auth-token-file FILE` | Node-generated mining authentication token file |
| `--tls-cert-sha256-file FILE` | Node-generated TLS certificate fingerprint file |
| `--tls-cert-sha256 HEX` | Certificate fingerprint as 64 hexadecimal characters, instead of the fingerprint file |

These options apply only to `-a quantus -s quic://HOST:PORT`. Use exactly
one certificate fingerprint option. Keep the authentication token private.

### GPU selection and tuning

| Command | Description |
| --- | --- |
| `-g`, `--gpu 0,1` | Mine only on the selected GPU indices |
| `--devices 0,1` | Alias for `--gpu` |
| `--list-gpus` | List NVIDIA GPU indices and exit |
| `--cuda-device 0,1` | CUDA-specific device selector |
| `--opencl-device 0,1` | OpenCL-specific device selector |
| `--opencl-platform N` | Select an OpenCL platform |
| `--cuda-disable` | Disable CUDA workers for an OpenCL run |
| `--opencl-enable` | Enable OpenCL mining for supported algorithms |
| `--no-cmp-unlock` | Skip automatic Linux CMP activation |
| `--cuda-no-blocking-sync` | Low-latency CUDA polling mode |
| `--cuda-spin-sync` | Lowest-latency CUDA mode with higher CPU usage |
| `--cuda-workload VALUE` | Set a manual CUDA workload for testing |
| `--autotune-cache FILE` | Store CUDA auto-tune profiles in a custom file |
| `--no-autotune-cache` | Do not load or save auto-tune results for this run |
| `--reset-autotune-cache` | Clear saved CUDA auto-tune profiles before starting |

CUDA workload tuning depends on the algorithm. Quantus uses its own bounded
batches; `--cuda-workload`, nonce-generator, and autotune-cache options do not
tune its backend. Do not assume legacy CUDA workload options control the
Equihash 192,7 solver either.

Optional NVIDIA hardware settings:

| Command | Description |
| --- | --- |
| `--gpu-core-clock MHZ[,MHZ...]` | Lock core clocks |
| `--gpu-memory-clock MHZ[,MHZ...]` | Lock memory clocks |
| `--gpu-power-limit W[,W...]` | Set power limits in watts |
| `--gpu-reset-tuning` | Restore locked clocks and default power limits before applying other tuning options |

Provide one value for all selected GPUs, or one value per selected GPU.
These options change GPU settings through NVML, may require administrator/root
permissions, and must be supported by the driver. There is no universal clock
or power preset for all cards. On HiveOS, prefer its standard rig controls.

### Quantus and Equihash settings

Leave these environment variables unset for normal automatic selection:

| Variable | Setting |
| --- | --- |
| `INTERMINER_QUANTUS_KERNEL` | Default: `checked` on SM86, `optimized` on other compatible GPUs |
| `INTERMINER_QUANTUS_BLOCK_SIZE` | Default: `128` |
| `INTERMINER_EQ192_BACKEND` | Default: `auto`; `legacy` explicitly selects the fallback solver |

The v1.2.8-6 packages also include the opt-in `checked-madd96` Quantus profile
for SM86 only (including RTX 3090). It requires block size `128` and remains
experimental: local kernel tests do not establish normal-worker or pool
validation. Do not enable it fleet-wide as a proven upgrade.

Windows CMD, before the normal Quantus command:

```bat
set INTERMINER_QUANTUS_KERNEL=checked-madd96
set INTERMINER_QUANTUS_BLOCK_SIZE=128
```

Restore defaults in CMD:

```bat
set INTERMINER_QUANTUS_KERNEL=
set INTERMINER_QUANTUS_BLOCK_SIZE=
```

Linux, in the same shell as the mining command:

```sh
export INTERMINER_QUANTUS_KERNEL=checked-madd96
export INTERMINER_QUANTUS_BLOCK_SIZE=128
```

Restore defaults with:

```sh
unset INTERMINER_QUANTUS_KERNEL INTERMINER_QUANTUS_BLOCK_SIZE
```

On HiveOS these are process environment variables, not command-line flags.
Do not paste `set` / `export` lines into the Custom Miner extra-config field.
The normal flight sheets below need no environment overrides.

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

1. Download and extract `InterMiner-v1.2.8-6-win64-amd64.zip`.
2. Edit the appropriate included `start-*.bat` file.
3. Set the wallet, worker name, pool, and GPU list.
4. Run the script.

Use `InterMiner-cuda.exe` for NVIDIA CUDA mining. Use `InterMiner.exe` with
`--cuda-disable --opencl-enable` for OpenCL mining.

## Linux

Requirements:

- x86-64 Linux with GLIBC 2.34 or newer for these release binaries.
- A compatible NVIDIA driver for CUDA mining.
- OpenCL loader and vendor drivers for AMD/OpenCL mining.

```bash
tar -xzf InterMiner-v1.2.8-6-linux-amd64.tar.gz
cd InterMiner-v1.2.8-6-linux-amd64
chmod +x InterMiner

LD_LIBRARY_PATH="$PWD:${LD_LIBRARY_PATH}" ./InterMiner \
  -a pearlhash \
  -s stratum+tcp://pearl-ru2.baikalmine.com:2010 \
  -w YOUR_WALLET.YOUR_WORKER \
  --password x --gpu 0
```

The standard `InterMiner` frontend loads the CUDA mining plugin for NVIDIA.
Linux/HiveOS packages include the small CUDA runtime; ordinary mining does
not require cuBLAS, cuBLASLt, or cuRAND. Keep the plugins next to the executable.

For Bitcoin BLAKE2b:

```bash
LD_LIBRARY_PATH="$PWD:${LD_LIBRARY_PATH}" ./InterMiner \
  -a blake2b -s stratum+tcp://stratum.minepoolis.com:4481 \
  -w YOUR_WALLET.YOUR_WORKER --password d=1 --gpu 0
```

For the new pool profiles:

```bash
LD_LIBRARY_PATH="$PWD:${LD_LIBRARY_PATH}" ./InterMiner \
  -a quantus -s stratum+tcp://ru.lproute.com:5660 \
  -w YOUR_QUANTUS_WALLET.YOUR_WORKER --gpu 0 --threads 0

LD_LIBRARY_PATH="$PWD:${LD_LIBRARY_PATH}" ./InterMiner \
  -a yec -s stratum+tcp://ninjaraider.com:44560 \
  -w YOUR_YEC_WALLET.YOUR_WORKER --gpu 0 --threads 0

LD_LIBRARY_PATH="$PWD:${LD_LIBRARY_PATH}" ./InterMiner \
  -a zcl -s stratum+tcp://equihash192.eu.mine.zpool.ca:2192 \
  -w YOUR_ZCL_WALLET.YOUR_WORKER --password c=ZCL,zap=ZCL --gpu 0 --threads 0
```

## HiveOS

Use this Custom Miner name:

```text
InterMiner-v1.2.8-6
```

Install URL:

```text
https://github.com/BaikalMine/InterMiner/releases/download/v1.2.8-6/InterMiner-v1.2.8-6-hiveos.tar.gz
```

Use a HiveOS system with GLIBC 2.34 or newer.

Supported miner algorithm values:

```text
cryptixhash
zelhash
pearlhash
sha3t
blake2b
quantus
ycash
zcl
```

Supply `-a` or `--algorithm` explicitly in the custom miner's extra config.
For BTCB2, use `-a blake2b --password d=1`, pool
`stratum.minepoolis.com:4481`, and wallet template `%WAL%.%WORKER_NAME%`.
`btcb2` is accepted as an alias for `blake2b`; `yec` is an alias for `ycash`.

For all examples use wallet template `%WAL%.%WORKER_NAME%` and set your own
wallet in the flight sheet:

| Coin | Pool URL | Extra config |
| --- | --- | --- |
| Quantus | `ru.lproute.com:5660` | `-a quantus --gpu 0` |
| YEC | `ninjaraider.com:44560` | `-a yec --gpu 0` |
| YEC on zpool | `equihash192.eu.mine.zpool.ca:2192` | `-a yec --password c=YEC,zap=YEC --gpu 0` |
| ZCL on zpool | `equihash192.eu.mine.zpool.ca:2192` | `-a zcl --password c=ZCL,zap=ZCL --gpu 0` |

Change `--gpu 0` to the required list, for example `--gpu 0,1`.
The zpool examples assume payouts in the named coin; change `c=` and the
wallet together when selecting a different supported payout currency.
Do not put the executable name in extra config; HiveOS starts the miner.

### PearlHash Flight Sheet JSON

The following text can be used as the PearlHash Custom Miner flight-sheet
configuration. Its wallet ID must exist in the target HiveOS account.

```json
{"name":"InterMiner","isFavorite":false,"items":[{"coin":"PRL","pool_ssl":false,"wal_id":11120435,"dpool_ssl":false,"miner":"custom","miner_alt":"InterMiner-v1.2.8-6","miner_config":{"url":"pearl-ru2.baikalmine.com:2010","miner":"InterMiner-v1.2.8-6","template":"%WAL%.%WORKER_NAME%","install_url":"https://github.com/BaikalMine/InterMiner/releases/download/v1.2.8-6/InterMiner-v1.2.8-6-hiveos.tar.gz","user_config":"-a pearlhash"},"pool_geo":[]}]}
```

## Developer Fee

| Algorithm | BaikalMine pools | Other pools |
| --- | ---: | ---: |
| CryptixHash | 0.75% | 1.0% |
| CS Coin ZelHash | 1.0% | 2.0% |
| PearlHash | 0.5% | 1.0% |
| SHA3T | 1.0% | 2.0% |
| Bitcoin BLAKE2b | 1.0% | 3.0% |
| Quantus pools | 0.5% | 1.0% |
| Ycash / YEC | 0.75% | 1.5% |
| Zclassic / ZCL | 0.75% | 1.5% |

Direct Quantus node mining has no developer fee.

PearlHash, SHA3T, Bitcoin BLAKE2b, Quantus pool, and YEC/ZCL fee work is
scheduled from accepted user shares. At 1%,
one fee share is scheduled per 100 accepted user shares. A rejected fee share
does not clear the outstanding fee work. BLAKE2b at 3% schedules three fee
shares per 100 accepted user shares. These are share-count rates, not a
guarantee of equal elapsed-time percentages when difficulties differ.
Fee shares are excluded from BLAKE2b user accepted/rejected statistics.

## Notes

- `Share accepted` confirms that the pool accepted a submitted share.
- The console shows the current hashrate and a 60-second average.
- ZelHash, YEC, and ZCL report solutions per second (`Sol/s`).
- Signed manifest verification and a content-addressed package cache are present
  as a foundation for future library updates. No remote package source is
  enabled by default.
