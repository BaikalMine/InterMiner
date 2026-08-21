# InterMiner for Keryx (KRX): Setup Guide

The latest Keryx network update requires support for the current PoM rules.
Mining requires **InterMiner 1.2.7 or newer**. Versions earlier than 1.2.3 do
not support H6, while earlier builds do not support the current network rules.

InterMiner 1.2.7 is compatible with the current Keryx update. The launch
command, algorithm, models, and direct-solo certificate remain unchanged. Pool
and solo-node operators must update Keryx node to **v1.5.1 or newer**.

PPLNS mining only requires a wallet address. Direct-solo additionally requires
a certificate that the address owner creates once in their wallet. The private
key is never transferred.

## 1. Requirements

| Item | Requirement |
| --- | --- |
| GPU | NVIDIA with compute capability 7.0 or newer (RTX 20/30/40/50 and compatible CMP cards) |
| VRAM | At least 8 GB for the minimum H6 profile |
| Driver | Current NVIDIA driver supporting the GPU and CUDA 12 |
| Free disk space | Approximately 10 GB |
| Miner version | **1.2.7 or newer** |

Current Keryx H6 mining is not supported on NVIDIA Pascal/GTX 10 or AMD/OpenCL.
For RTX 50 cards, use a recent driver with Blackwell support.

Check the driver on Windows or Linux:

```text
nvidia-smi
```

On the first start, the miner downloads the current language model, which is
approximately 6.5 GB. This is expected and normally happens only once.

## 2. PPLNS

Regular PPLNS mining does not require a certificate.

Linux:

```bash
LD_LIBRARY_PATH="$PWD:${LD_LIBRARY_PATH}" ./InterMiner-cuda \
  -a keryxhash \
  -s stratum+tcp://krx.baikalmine.com:9020 \
  -w keryx:YOUR_ADDRESS.rig01 \
  --threads 0 --cuda-no-blocking-sync
```

Windows:

```bat
InterMiner-cuda.exe -a keryxhash -s stratum+tcp://krx.baikalmine.com:9020 -w keryx:YOUR_ADDRESS.rig01 --threads 0 --cuda-no-blocking-sync
```

`rig01` is an arbitrary worker name. `--threads 0` disables CPU mining.

## 3. Direct Solo

In direct-solo mode, the block reward is assigned directly to your address.
After H6, the block must prove that the miner's local escrow key is authorized
by the owner of that address. The pool cannot create this certificate on behalf
of the address owner.

Important: use a wallet address whose private key you control. An exchange,
custodial service, or other wallet address for which you do not control the
private key cannot be used for H6 direct-solo mining.

### Step 1. Get the miner escrow key

Start the miner without a certificate.

Linux:

```bash
LD_LIBRARY_PATH="$PWD:${LD_LIBRARY_PATH}" ./InterMiner-cuda \
  -a keryxhash \
  -s stratum+tcp://krx-solo.baikalmine.com:9021 \
  -w keryx:YOUR_ADDRESS.rig01 \
  --threads 0 --cuda-no-blocking-sync
```

Windows:

```bat
InterMiner-cuda.exe -a keryxhash -s stratum+tcp://krx-solo.baikalmine.com:9021 -w keryx:YOUR_ADDRESS.rig01 --threads 0 --cuda-no-blocking-sync
```

Use the exact host name `krx-solo.baikalmine.com`; InterMiner uses it to detect
direct-solo mode. On the first start, the miner creates `escrow.key`, stops, and
prints:

```text
H6 escrow key to authorize in the payout wallet: <64 hex>
```

Copy the 64 hexadecimal characters after the colon.

### Step 2. Authorize the key

Open the Keryx wallet that owns the specified payout address, select the miner
authorization function, and sign the escrow key from step 1. The wallet returns
a certificate containing 128 hexadecimal characters. The certificate is
public; your private key and seed phrase are never transferred.

### Step 3. Pass the certificate to the miner

For the first start with the certificate, use:

```text
--escrow-cert <128 hex>
```

Complete Windows example:

```bat
InterMiner-cuda.exe -a keryxhash -s stratum+tcp://krx-solo.baikalmine.com:9021 -w keryx:YOUR_ADDRESS.rig01 --threads 0 --cuda-no-blocking-sync --escrow-cert YOUR_CERTIFICATE
```

InterMiner validates the certificate before connecting and saves it as
`escrow.cert`. You can remove `--escrow-cert` from subsequent starts. To use an
existing file directly, specify:

```text
--escrow-cert-file <path to escrow.cert>
```

On HiveOS, `escrow.key`, `escrow.cert`, and escrow state are stored in
`/hive/miners/custom/interminer-state/` by default so they survive miner
updates.

## 4. Changing the Address or Reinstalling

The certificate is bound to the payout address and miner escrow key pair. You
need a new certificate if the payout address changes or `escrow.key` is lost.
Back up `escrow.key` with the rig configuration. Never send this file to anyone.

## 5. Common Errors

### Outdated miner version

An unsupported miner message after a network update means that the installed
version is outdated. The current network requires version 1.2.7 or newer.
Update the miner itself; changing the command line will not fix this error.

### Missing direct-solo certificate

```text
H6 direct solo requires a delegation certificate
```

Repeat the steps in section 3.

### Certificate does not match the address

```text
escrow certificate does not match the payout address and escrow key
```

The certificate was created for a different payout address or a different
`escrow.key`. Create a new certificate for the key printed by the miner during
the current start.

### Corrupted model

If `failed verification` appears after a complete model check, delete the model
file and its `.ok` marker, then start the miner again:

```text
<model directory>/Qwen3.5-9B-abliterated/model.gguf
<model directory>/Qwen3.5-9B-abliterated/.ok
```

Model directory locations:

| Launch mode | Directory |
| --- | --- |
| Regular | `models/` next to InterMiner |
| HiveOS | `/hive/miners/custom/interminer-models/` |
| With `--model-dir DIR` | The specified `DIR` |

On HiveOS, the model directory is outside the version-specific miner directory,
so it is retained during upgrades. InterMiner also prints the selected model
directory near the beginning of the log.

## 6. Pool Addresses

| Mode | Address | User certificate |
| --- | --- | --- |
| PPLNS | `stratum+tcp://krx.baikalmine.com:9020` | Not required |
| Direct solo | `stratum+tcp://krx-solo.baikalmine.com:9021` | Required |

## 7. Useful Options

| Option | Purpose |
| --- | --- |
| `--list-gpus` | List GPUs and exit |
| `--gpu 0,2` | Mine only on the selected GPUs |
| `--threads 0` | Disable CPU mining |
| `--password VALUE` | Pool password, when required |
| `--escrow-cert <128 hex>` | Pass a certificate as text and save it |
| `--escrow-cert-file FILE` | Read a certificate from a file |
| `--hiveos` | Use persistent HiveOS directories for models and state |
| `--model-dir DIR` | Use a dedicated model directory |
| `--version` | Print the miner version |
| `--help` | Print all available options |
