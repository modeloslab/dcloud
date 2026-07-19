# How dCloud works

## The big picture

dCloud is a two-sided network:

- **Providers** (you) run the `dcloud-plugin` node on a GPU machine.
- **Users** submit paying workloads (inference requests, training runs, rentals) to the **coordinator**
  at `https://dcloud.modeloslab.xyz`.

The coordinator matches workloads to capable providers, streams the work to your node, verifies the
result, and settles the reward. Everything is driven from the **Provider console** — you never edit
config files to change what your machine does.

```
   You (GPU node)                 Coordinator                    Users
 ┌────────────────┐        ┌──────────────────────┐        ┌────────────┐
 │  dcloud-plugin │◄──────►│  dcloud.modeloslab   │◄──────► │  inference │
 │  (native bin)  │  pair  │  .xyz                 │  jobs  │  training  │
 │      + Python  │  jobs  │  match • verify •     │        │  rentals   │
 │      sidecar   │  proof │  settle rewards       │        │            │
 └────────────────┘        └──────────────────────┘        └────────────┘
```

## The node: one binary + a self-installing runtime

- The **native binary** (`dcloud-plugin`, written in Rust) handles networking, pairing, the coordinator
  protocol, GPU detection, and supervising the workload.
- On the **first launch** of any Python-backed service, it provisions a **private Python virtual
  environment** (pinned PyTorch / Transformers, CUDA build auto-selected from your driver, Metal on
  macOS). This is a one-time download (~2–5 GB) cached under `~/.modelos/dcloud-py`. No manual pip.
- The **Python sidecar** (bundled as source in `python/modelos_dcloud/`) runs the actual model
  inference / training / sharding.

## Hardware is auto-detected, never declared

When the node starts it runs `nvidia-smi` (NVIDIA) or queries Metal (Apple Silicon) and reports the
**real** GPU. You cannot declare a fake GPU in config — this keeps the "verified cloud" honest. If no
GPU is found, GPU services refuse to start.

## The five services

- **dHost** — your GPU serves a full model; requests arrive via the network's OpenAI-compatible
  inference API and your node answers them.
- **dShard** — for models too big for one GPU (e.g. 70B+), the coordinator splits the model across
  several nodes into a pipeline **ring**; your node serves an assigned slice of the layers and streams
  activations to the next node. Joining or leaving re-partitions the ring automatically.
- **dTrain** — your node joins a distributed training run (DiLoCo-style), doing local optimizer steps
  and contributing updates.
- **dStorage** — your node provides verified storage capacity.
- **dCloud (rental)** — you rent the raw GPU by the epoch; renters run their own containers/workloads.

Enable any combination and split your GPU between them from the console (percentage allocation).

## Verification & rewards

Work is verified by the network (proofs / attestation depending on the service), and rewards settle to
your paired worker identity. You control that identity with a **worker key** (`DCLOUD_WORKER_KEY`) — the
same key always produces the same worker address, so your machine is stable across restarts. Pairing a
machine in the console links it to your account for control and payout.

## Pairing & control

On `serve`, the node prints:

- a **worker address** (`mdl1p…`) — your machine's network identity, and
- a **pairing code** — a token that authorizes the console to drive this machine.

Enter both in the Provider console once. From then on you switch services on/off, pause, set the model,
and see live utilization — all remotely, without touching the machine.
