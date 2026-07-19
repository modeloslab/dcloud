# Join the network

Once the node is installed ([install.md](install.md)), joining is three steps: **run → pair → enable**.

## 1. Run the node

Start it idle (no services yet) so you can turn things on from the console:

- macOS / Linux:
  ```bash
  ./run.sh serve --manifest-json '{"mode":"percent","services":[]}'
  ```
- Windows:
  ```bat
  run.bat serve --manifest-json "{\"mode\":\"percent\",\"services\":[]}"
  ```

> Tip: set a stable identity first so restarts keep the same address:
> `export DCLOUD_WORKER_KEY=<your 64-hex key>` (macOS/Linux) or
> `set DCLOUD_WORKER_KEY=<your 64-hex key>` (Windows). Save the key somewhere safe.

## 2. Pair it in the console

The node prints two things when it starts:

```
worker address (enter this in the dCloud console): mdl1p...............................
control paired — enter this pairing code in the dCloud console to drive this machine: a1b2c3...
```

1. Open the **Provider console**: **https://dcloud.modeloslab.xyz**
2. Sign in / create your account.
3. Add a machine → paste the **worker address** and the **pairing code**.
4. Your machine appears as **online / idle** with its auto-detected GPU. It's now controllable
   remotely.

## 3. Enable services

From the console, flip on whatever you want your GPU to do and set each service's share of the GPU:

- **dHost** — serve a full model for inference. Pick the model in the console.
- **dShard** — join a sharded ring for large (70B+) models; the coordinator assigns your slice
  automatically. When more nodes join, the ring re-partitions on its own.
- **dTrain** — join a distributed training run.
- **dStorage** — provide storage.
- **dCloud (rental)** — list your raw GPU for rent by the epoch.

You can run several at once (e.g. 60% dShard / 40% dHost) — the console controls the split live. Pause
or switch services any time without restarting the node.

## 4. Get paid

Work your node completes is verified by the network and rewards settle to your paired worker identity
(the address you paired in step 2, derived from your `DCLOUD_WORKER_KEY`). Keep the node running and
paired; the console shows live utilization and status.

## Keeping it running

- **Leave it up.** The node reconnects automatically and re-joins rings after restarts. Because the
  binary caches its runtime and (for sharded models) its quantized weights on disk, restarts and
  re-partitions come back up in seconds, not minutes.
- **Run as a service** (optional) so it survives reboots:
  - Linux: wrap `run.sh` in a `systemd` unit.
  - macOS: a `launchd` plist.
  - Windows: Task Scheduler → *At startup*.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `no GPU detected … providers require a real GPU` | Install/enable your NVIDIA driver (`nvidia-smi` must work), or run on Apple Silicon. dCloud needs a real GPU. |
| First launch hangs on "provisioning runtime" | It's downloading the ML deps once (~5 GB dShard/dTrain, ~10–15 GB dHost/SGLang). Give it time; later launches are instant. |
| First launch **fails mid-download** (network timeout / partial download) | Expected on slow/interrupted connections for a download this size. Just **run the same command again** — uv resumes and the pinned lockfile reinstalls the exact same versions. Confirm you have the free disk space. |
| macOS "cannot be opened" / Gatekeeper | `xattr -dr com.apple.quarantine ./dcloud-plugin` |
| Windows SmartScreen warning | *More info → Run anyway* (unsigned binary). |
| Machine not appearing in console | Re-check the worker address + pairing code were pasted exactly; confirm the node still shows them in its logs. |
| dHost fails on first launch with `ninja exited with status 1` / a compiler error | dHost JIT-compiles a CUDA kernel on first serve and needs a host C++ compiler. Install it: `sudo apt-get install -y build-essential` (Debian/Ubuntu) or the `gcc-c++` / "Development Tools" group (RHEL/Fedora), then relaunch. Only dHost needs this. |
| Python errors on start | Ensure Python 3.11+ is on PATH; delete `~/.modelos/dcloud-py` to force a clean runtime reinstall. |
