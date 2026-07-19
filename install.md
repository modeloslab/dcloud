# Install

The node ships as a single native binary plus a bundled Python sidecar. The **first launch** installs
its own Python dependencies into a private virtual environment — you do **not** run pip yourself.

## Prerequisites (all platforms)

- **Python 3.11** (exact — the sidecar is compiled for CPython 3.11) on your PATH (`python3 --version` / `python --version`).
- A **GPU**:
  - **NVIDIA**: recent driver + `nvidia-smi` on PATH (CUDA is used automatically). You do **not** need the
    CUDA toolkit — the node bundles the compiler pieces (`nvcc`/CCCL/runtime) it needs.
  - **Apple Silicon** (M1–M4): nothing extra — Metal is used automatically.
- **dHost only (NVIDIA):** a host **C++ compiler** (`g++`). dHost's deterministic inference engine JIT-compiles a
  small CUDA kernel on first launch; the node ships the CUDA toolkit but relies on the system `g++`. Install once:
  `sudo apt-get install -y build-essential` (Debian/Ubuntu) or the `gcc-c++`/"Development Tools" group (RHEL/Fedora).
  Not needed for dShard/dStorage/dCloud/dTrain.
- Free disk for the runtime (~2–5 GB) plus any models you serve.

> No GPU? The node will exit on GPU services by design — dCloud only lists real, verified hardware.

---

## macOS — Apple Silicon (M1 / M2 / M3 / M4)

```bash
# 1. Download + extract
curl -LO https://github.com/modeloslab/dcloud/releases/latest/download/dcloud-plugin-macos-arm64.tar.gz
tar -xzf dcloud-plugin-macos-arm64.tar.gz
cd dcloud-plugin-macos-arm64

# 2. (macOS Gatekeeper) allow the unsigned binary the first time
xattr -dr com.apple.quarantine ./dcloud-plugin

# 3. Run (idle to start — you enable services from the console)
./run.sh serve --manifest-json '{"mode":"percent","services":[]}'
```

## macOS — Intel

Same as Apple Silicon, using the Intel bundle:

```bash
curl -LO https://github.com/modeloslab/dcloud/releases/latest/download/dcloud-plugin-macos-x64.tar.gz
tar -xzf dcloud-plugin-macos-x64.tar.gz
cd dcloud-plugin-macos-x64
xattr -dr com.apple.quarantine ./dcloud-plugin
./run.sh serve --manifest-json '{"mode":"percent","services":[]}'
```

## Linux (x86-64)

```bash
# 1. Download + extract
curl -LO https://github.com/modeloslab/dcloud/releases/latest/download/dcloud-plugin-linux-x64.tar.gz
tar -xzf dcloud-plugin-linux-x64.tar.gz
cd dcloud-plugin-linux-x64

# 2. Ensure Python 3.11+ (Ubuntu example)
sudo apt-get update && sudo apt-get install -y python3 python3-venv

# 3. NVIDIA driver must be present (check):
nvidia-smi

# 4. Run
./run.sh serve --manifest-json '{"mode":"percent","services":[]}'
```

## Windows (x64)

1. Download `dcloud-plugin-windows-x64.zip` and extract it (right-click → Extract All).
2. Install **Python 3.11** (exact — the sidecar is compiled for CPython 3.11) from python.org (tick *"Add Python to PATH"*).
3. Ensure your **NVIDIA driver** is installed (`nvidia-smi` should work in a terminal).
4. Open **PowerShell** or **Command Prompt** in the extracted folder and run:

```bat
run.bat serve --manifest-json "{\"mode\":\"percent\",\"services\":[]}"
```

> Windows may prompt via SmartScreen the first time — choose *More info → Run anyway* for the unsigned
> binary.

---

## First launch: what to expect

The first time you start a GPU service, the node provisions a self-contained Python runtime for that service
using **[uv](https://docs.astral.sh/uv/)** with a **pinned lockfile** — so every provider installs the exact,
pre-resolved dependency set (no version drift, no resolver surprises):

```
dCloud FIRST-LAUNCH bootstrap (venv-shard): provisioning a self-contained Python runtime …
uv: installed Python 3.11 + synced the pinned lockfile
```

This is normal and happens once per service; later launches skip straight to serving.

> **Heads-up — this download is large and network-dependent.** The ML stack is big (roughly **5 GB** for
> dShard/dTrain, and **10–15 GB** for the dHost SGLang engine). **A slow or interrupted connection can fail the
> download mid-way.** If the first launch errors out (network timeout / partial download), it is safe to just
> **run it again** — uv resumes and is idempotent, and the pinned lockfile means the retry installs the exact
> same versions. Make sure you have the free disk space (see Prerequisites) before starting. Each service uses
> its **own** isolated environment, so a node running both dHost and dShard downloads both stacks.

## Useful environment variables

| Variable | Purpose | Default |
|----------|---------|---------|
| `DCLOUD_WORKER_KEY` | Your stable worker identity (same key → same address). **Set this** so your machine is consistent across restarts. | random each run |
| `DCLOUD_COORDINATOR_URL` | Network coordinator | `https://dcloud.modeloslab.xyz` |
| `DCLOUD_PY_HOME` | Where the private venv is installed | `~/.modelos/dcloud-py` |
| `DCLOUD_PYTHON` | Use an existing Python interpreter instead of bootstrapping one | (bootstrap) |

Example (Linux/macOS): pin a stable identity —

```bash
export DCLOUD_WORKER_KEY=$(openssl rand -hex 32)   # save this; it is your machine's identity
./run.sh serve --manifest-json '{"mode":"percent","services":[]}'
```

Next: **[Join the network](join-network.md)**.
