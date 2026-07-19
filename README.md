# dCloud — modelOS decentralized verified AI cloud

**Turn your GPU into income.** Run one binary, pair it in the console, and your machine joins the
modelOS dCloud network — serving AI inference, sharded large models, distributed training, storage, and
raw GPU rental. Hardware is auto-detected and verified; you get paid for the work your GPU does.

> **Repo description:** One-binary provider node for the modelOS dCloud network — serve AI inference,
> sharded 70B+ models, distributed training, and GPU rental from Mac, Linux, or Windows.

---

## What is dCloud?

dCloud is a network of independent GPU providers coordinated into a single **verified** AI cloud. A
central coordinator matches paying workloads to your machine and settles rewards on-chain; your node
proves the work it did. Five services run from the same binary — enable any mix from the web console:

| Service | What your GPU does |
|---------|--------------------|
| **dHost**   | Serve a whole model for inference (OpenAI-compatible API) |
| **dShard**  | Serve *part* of a large model (70B+) in a cross-node pipeline ring |
| **dTrain**  | Contribute to distributed (DiLoCo) training runs |
| **dStorage**| Provide verified storage |
| **dCloud**  | Rent your raw GPU by the epoch (renters run any workload) |

You never declare your hardware — the node auto-detects your GPU (NVIDIA via `nvidia-smi`, Apple
Silicon via Metal) so the network stays honest.

## Quick start (60 seconds)

1. **Download** the bundle for your OS from the [**Releases**](https://github.com/modeloslab/dcloud/releases/latest) page.
2. **Extract** it.
3. **Run** it:
   - macOS / Linux: `./run.sh serve --manifest-json '{"mode":"percent","services":[]}'`
   - Windows: `run.bat serve --manifest-json "{\"mode\":\"percent\",\"services\":[]}"`
4. **Pair** it — the node prints a **worker address** and a **pairing code**. Enter both in the
   Provider console at **https://dcloud.modeloslab.xyz**.
5. **Enable services** in the console (e.g. flip on dHost or dShard). That's it — you're serving.

The very first run provisions a self-contained Python runtime automatically (one-time download); see
[install.md](install.md).

## Documentation

- **[How it works](how-it-works.md)** — architecture, services, verification, rewards
- **[Install](install.md)** — step-by-step for macOS (Apple Silicon + Intel), Linux, Windows
- **[Join the network](join-network.md)** — pairing, enabling services, and getting paid

## Requirements

- A **GPU**: NVIDIA (with driver + `nvidia-smi`) for CUDA nodes, or Apple Silicon (Metal). No GPU → the
  node exits by design (the network only lists real, verified hardware).
- **Python 3.11+** on your PATH (the node installs its own ML dependencies into a private venv).
- Disk for the runtime + any models it serves (~2–5 GB runtime; models vary).

## Downloads

Grab the latest bundle for your OS from the [**Releases**](https://github.com/modeloslab/dcloud/releases/latest) page:

| Platform | File |
|----------|------|
| macOS (Apple Silicon, M1–M4) | `dcloud-plugin-macos-arm64.tar.gz` |
| macOS (Intel) | `dcloud-plugin-macos-x64.tar.gz` |
| Linux (x86-64) | `dcloud-plugin-linux-x64.tar.gz` |
| Windows (x64) | `dcloud-plugin-windows-x64.zip` |

Each bundle contains the native `dcloud-plugin` binary, the Python sidecar, a launcher, and a README.
Verify your download against `SHA256SUMS.txt` (also on the Releases page).
