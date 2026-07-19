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
InterMiner.exe --keryxd-address stratum+tcp://krx.baikalmine.com:9020 --threads 0 --mining-address keryx:YOUR_WALLET.YOUR_WORKER
```

### CryptixHash v2

Use a Cryptix stratum URL and `cryptix:` wallet. OPoI is disabled for this profile.

```bat
InterMiner.exe --cryptix-address stratum+tcp://cytx.baikalmine.com:9010 --mining-address cryptix:YOUR_WALLET.YOUR_WORKER
```

Use the included `start-*.bat` script for the chosen profile; edit only its pool and wallet fields.

## Windows

1. Download and extract the Windows ZIP.
2. Edit the matching start script.
3. Set your wallet and worker name.
4. Run the script.

A current NVIDIA driver is required for CUDA profiles. Full packages include CUDA runtime libraries; light packages use compatible libraries installed on the system.

## Linux

```bash
tar -xzf interminer-v1.1.0-linux-amd64-cuda.tar.gz
cd interminer-v1.1.0-linux-amd64-cuda
chmod +x InterMiner
LD_LIBRARY_PATH="$PWD:${LD_LIBRARY_PATH}" ./InterMiner --help
```

## HiveOS

Install the HiveOS archive through a Custom Miner flight sheet. The `Name` / `miner_alt` value must exactly match the root folder inside the archive; the release notes provide that value and the installation URL.

## Developer Fee

- BaikalMine pools (`*.baikalmine.com`): 1.5%
- Other supported pools: 3%

## Notes

- Release archives contain binaries only; source code and OPoI models are not bundled.
- Keep `escrow.key` private when mining Keryx OPoI.
- The terminal dashboard shows per-GPU speed and telemetry when exposed by the driver.
- Run `InterMiner --help` for profile and tuning options.