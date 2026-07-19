# InterMiner

InterMiner is a GPU miner with profile-based support for KeryxHash / OPoI and CryptixHash v2. This repository contains release packages only. Source code is maintained in [BaikalMine-Pools/b-miner](https://github.com/BaikalMine-Pools/b-miner).

## Download

Download the current release from the [Releases page](https://github.com/BaikalMine/InterMiner/releases). Packages are published for Windows, Linux, and HiveOS.

## Supported GPUs

- NVIDIA GTX 10 series and newer
- NVIDIA RTX 20, 30, 40, and 50 series
- AMD RX 6000 series and newer where the selected profile provides OpenCL support

## Profiles

### KeryxHash / OPoI

Use a Keryx stratum URL and `keryx:` wallet. OPoI models are selected from GPU VRAM and verified before use.

```bat
InterMiner.exe --threads 0 -s stratum+tcp://krx.baikalmine.com:9020 -a keryx:YOUR_WALLET.YOUR_WORKER
```

### CryptixHash v2

Use a Cryptix stratum URL and `cryptix:` wallet. OPoI is disabled for this profile.

```bat
InterMiner.exe --algorithm cryptixhash --threads 0 -s stratum+tcp://cytx.baikalmine.com:9010 -a cryptix:YOUR_WALLET.YOUR_WORKER
```

Use the included `start-*.bat` script for the chosen profile; edit only its pool and wallet fields.

## Windows

1. Download and extract the Windows ZIP.
2. Edit the matching start script.
3. Set your wallet and worker name.
4. Run the script.

A current NVIDIA driver is required for CUDA profiles. The 1.1.0 release packages are full packages and include the CUDA runtime libraries needed by the bundled CUDA backend.

## Linux

```bash
tar -xzf InterMiner-v1.1.0-linux-amd64.tar.gz
cd InterMiner-v1.1.0-linux-amd64
chmod +x InterMiner
LD_LIBRARY_PATH="$PWD:${LD_LIBRARY_PATH}" ./InterMiner --help
```

## HiveOS

Install the `InterMiner-v1.1.0-hiveos.tar.gz` archive through a Custom Miner flight sheet. Set `Name` and `miner_alt` to `InterMiner-v1.1.0`, exactly matching the archive root folder.

## Developer Fee

- BaikalMine pools (`*.baikalmine.com`): 1.5%
- Other supported pools: 3%

## Notes

- Release archives contain binaries only; source code and OPoI models are not bundled.
- Keep `escrow.key` private when mining Keryx OPoI.
- The terminal dashboard shows per-GPU speed and telemetry when exposed by the driver.
- Run `InterMiner --help` for profile and tuning options.
