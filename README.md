# BtcMole — World's Fastest Bitcoin Puzzle Solver (as of March 2026)

To the best of the author's knowledge, BtcMole is the world's
fastest program for solving Bitcoin puzzles given only the P2PKH
address. On a single NVIDIA GeForce RTX 5090 it sustains
**over 9.5 billion keys per second** in long-running puzzle search.

BtcMole is a tool for solving Bitcoin puzzles (#71 and higher) and for
recovering forgotten private keys when you know the key range and your
own P2PKH address.

GPU acceleration — NVIDIA via CUDA and AMD via ROCm / HIP — is
available on **Linux x86-64** and **Windows x86-64**. CPU-only
operation is supported on every platform: **Linux x86-64**,
**Windows x86-64**, **macOS arm64**, **Linux arm**, **Linux arm64**,
and **Android through Termux**. Pre-built binaries for every
supported platform are published in this repository as ZIP
archives.

## Quick start

```
# 1. Solve Bitcoin Puzzle #71 using every available CPU and GPU.
bm072 bf -pz 71 +cpu +cuda +amdgpu

# 2. Recover a forgotten wallet — known address, known approximate range.
bm072 bf --address 1XYZ... --range 100000:1FFFFF +cpu +cuda

# 3. Dig in the public pool and earn pool-time.
#    (Launch the @BtcMoleBot mini-app to obtain a session number.)
bm072 dig -s <session>

# 4. Full throttle — use 100% of CPU threads and every GPU compute
#    unit. By default the program leaves a small reserve so the
#    machine stays usable for other tasks during the search.
bm072 dig -s <session> +cpu:100% +cuda:100% +amdgpu:100%
```

On Linux / macOS prepend `./` to the binary name (`./bm072 ...`).
Substitute your actual release number for `072` (see
[A note on the executable name](#a-note-on-the-executable-name)).

## Features

- Highly optimised secp256k1 modular arithmetic for CPU (amd64, arm64, arm) and GPU (NVIDIA, AMD).
- Full GPU stack out of the box. A single command can drive cards
  from both vendors and the CPU at the same time.
    - **NVIDIA** via CUDA — compute capability 5.0 and up.
    - **AMD on Linux** — every architecture from **GCN 3-4** (R9
      Fury and the Polaris RX 400 / RX 500 series) through
      **GCN5 / Vega**, **CDNA 1-3**, and **RDNA 1-4** is supported.
    - **AMD on Windows** — **RDNA2** (RX 6000 series) through
      **RDNA4** (RX 9000 series) under AMD's current HIP SDK.
      **Radeon VII / Vega 20** has worked under some older HIP SDK
      versions — your mileage may vary.
- Per-class and per-card tuning: `+cuda:N`, `+amdgpu:50%`,
  per-card lists, reserving compute units for the desktop GPU,
  capping CPU thread counts.
- Resumable bruteforce. Every interrupted run picks up exactly where
  it stopped: per-CPU and per-GPU state files are written atomically
  next to a progress map.
- Random sector selection in offline search, so that different users
  running the program independently are unlikely to retread the same
  key ranges.
- Two modes — fully offline `bf` (bruteforce on your own hardware)
  and online `dig` (pool mode, server-distributed work).
- Built-in self-update via `bmXXX upgrade`.
- Ready-to-run binaries for Linux x86-64, Windows x86-64, Linux arm /
  arm64, macOS arm64, and Android through Termux. No build toolchain,
  no driver SDKs, no runtime dependencies beyond the GPU driver.

## Performance

Real key-search throughput, measured by running the program on actual
Bitcoin Puzzle targets — not synthetic benchmarks.

| Device                   | Architecture        |   Speed (keys/s) |
|--------------------------|---------------------|------------------|
| NVIDIA GeForce RTX 5090  | Blackwell (SM_120)  |    9 523 Mkey/s  |
| NVIDIA GeForce RTX 5080  | Blackwell (SM_120)  |    4 551 Mkey/s  |
| NVIDIA CMP 90HX          | Ampere (SM_86)      |    1 912 Mkey/s  |
| AMD Radeon R9 Fury      | GCN3                      |      224 Mkey/s  |

All figures are sustained long-run throughput.

More user-contributed numbers for individual cards are collected in
the Telegram user group — see the link below.

## Device selection tips

- **Ryzen APU integrated GPUs** (Vega-class and RDNA2-class iGPUs
  baked into modern Ryzen processors) are detected as regular AMD
  cards and work fine. There is no point in running `+cpu` together
  with the iGPU on the same chip — they share one power budget, so
  the CPU side will only steal watts and add little to the total
  speed. Use `+amdgpu -cpu`.
- **If you have a discrete NVIDIA card**, you usually do not need
  the CPU at all. A modern CUDA GPU is dramatically more
  energy-efficient than any CPU on this workload — adding `+cpu`
  alongside `+cuda` typically buys a single-digit-percent speedup
  while consuming a lot of extra wall power. `+cuda -cpu` is almost
  always the right default.

## Which build to download

Pre-built archives live in three OS folders at the root of this
repository:

- [`linux/`](linux/) — Linux builds (x86-64 + arm / arm64).
- [`windows/`](windows/) — Windows x86-64 builds.
- [`macos/`](macos/) — macOS arm64 build.

For Linux x86-64 and Windows x86-64 there are four ZIPs in every
release; they differ in which GPU back-ends are compiled into the
binary:

| Suffix          | NVIDIA | AMD | When to use it                                             |
|-----------------|--------|-----|------------------------------------------------------------|
| (no suffix)     | no     | no  | CPU-only machine, or you do not want any GPU code at all.  |
| `_cuda`         | yes    | no  | The machine has NVIDIA GPU(s) only.                        |
| `_amdgpu`       | no     | yes | The machine has AMD GPU(s) only.                           |
| `_gpu`          | yes    | yes | Mixed NVIDIA + AMD machine (or you don't know in advance). |

Example: on a Linux box with an NVIDIA RTX card take
[`linux/bm072.linux_amd64_cuda.zip`](linux/bm072.linux_amd64_cuda.zip);
on a Linux box with an AMD card take
[`linux/bm072.linux_amd64_amdgpu.zip`](linux/bm072.linux_amd64_amdgpu.zip);
on a Linux box with both — take
[`linux/bm072.linux_amd64_gpu.zip`](linux/bm072.linux_amd64_gpu.zip).

The CPU-only build
([`linux/bm072.linux_amd64.zip`](linux/bm072.linux_amd64.zip) /
[`windows/bm072.windows_amd64.zip`](windows/bm072.windows_amd64.zip))
is the smallest and has no GPU runtime requirements at all — useful on
servers / VPS with no GPU. The GPU builds also work fine without a
GPU (they just fall back to CPU), but they are larger because they
embed precompiled GPU kernels.

For non-amd64 platforms ([Linux arm](linux/bm072.linux_arm.zip),
[Linux arm64](linux/bm072.linux_arm64.zip),
[macOS arm64](macos/bm072.darwin_arm64.zip)) only the CPU build is
published — there is no `_cuda` / `_amdgpu` / `_gpu` variant for
those.

## A note on the executable name

Every release of the program ships with the version number baked into
the executable file name: `bmXXX`, where `XXX` is the three-digit
version. For example, the current release
[`linux/bm072.linux_amd64.zip`](linux/bm072.linux_amd64.zip) unpacks
to an executable called `bm072`. The next release will be
named `bm073`, then `bm074`, and so on.

In the command examples below the program is therefore invoked as
`bmXXX` — substitute the actual version of the binary you have
downloaded. For instance, if you are using release 072 the commands
look like `bm072 bf ...` and `bm072 dig ...`.

The exact form of the command also depends on your operating system:

- On **Linux** and **macOS** the executable in the current directory
  is invoked with a leading `./`, e.g. `./bm072 bf -pz 71`.
- On **Windows** the leading `./` is not used; the command is simply
  `bm072 bf -pz 71` (or `bm072.exe bf -pz 71`).

The examples in the rest of this document are written without `./`
for brevity; add it on Linux/macOS.

## Android

There is no native Android build, but the Linux arm/arm64 binaries
work inside [Termux](https://termux.dev/). The exact step-by-step
instructions for setting Termux up and running BtcMole on a phone
are kept up to date in the user group — start here:
[https://t.me/c/2550754303/425/451](https://t.me/c/2550754303/425/451)
(or check the Telegram channel for the latest pinned message).

## Two modes of operation

### 1. Bruteforce mode (`bmXXX bf`)

Offline search across a key range on your own hardware. Two ways to
specify the target:

- `bmXXX bf -pz N` — search for Bitcoin puzzle number `N` (any `N` in
  1..160). The address and the key range for the puzzle are built in.
- `bmXXX bf --address <P2PKH> --range <start:end>` — search for an
  arbitrary P2PKH address inside an arbitrary hex key range.

You can mix device classes freely on the same task, for example
`+cpu +cuda +amdgpu` to engage every available CPU core and every
detected GPU at the same time. Per-class fine tuning is available
(`+cuda:N`, `+amdgpu:50%`, per-card lists, etc.).

How bruteforce mode works:

- The chosen key range is split into sectors. A progress map file
  (`bf_<id>.map`) is created next to the binary.
- Every running manager (one per CPU pool, one per GPU card) claims a
  free sector from the shared map, processes it, and marks the sector
  as done. Several cards on the same machine cooperate through this
  map without overlapping work.
- Sectors are picked at random rather than sequentially. This is done
  so that different users running the program independently do not
  end up doing the same work — without coordination, each instance
  is unlikely to repeat key ranges already searched by someone else.
- Progress is checkpointed to disk. On `Ctrl+C`, crash, or reboot the
  next launch resumes from the saved state of each device — the
  current sector continues from the exact point it was interrupted at.
- When a key is found, the result is saved to a `FOUND_*.txt` file in
  the working directory and the run stops.

This mode is fully offline — no network access is required after the
binary is downloaded.

### 2. Pool mode (`bmXXX dig`)

Online cooperative search coordinated by a server. Many participants
work on the same puzzle in parallel; each client receives work
assignments from the server and reports back.

To use pool mode you need to register in the Telegram bot
[@BtcMoleBot](https://t.me/BtcMoleBot) and follow the instructions it
gives you (you will receive a session identifier that you pass to the
program with `-s <session>`).

Announcements and progress updates are posted in the Telegram channel
[https://t.me/BtcMole](https://t.me/BtcMole).

There is also a Telegram group for users of the program where you can
ask questions, share experience and discuss settings with other
participants:
[https://t.me/+AIPjS1v-u3FlZjZi](https://t.me/+AIPjS1v-u3FlZjZi).

If you run into trouble with pool mode (registration, session,
payouts, anything that does not work as expected) please contact the
support account on Telegram: [@BtcMoleSup](https://t.me/BtcMoleSup).

## After a key is found

When the program locates the private key for the target address, it
writes a result file in the current working directory and stops the
run:

- `FOUND_PUZZLE_<N>.txt` — for puzzle searches (`bf -pz N` and `dig`).
- `FOUND_KEY_<address>.txt` — for arbitrary-target searches
  (`bf --address ...`).

If you want to see in advance what a successful find looks like —
both the console output and the contents of the result file — run
the program once against a small Bitcoin puzzle whose answer is
already public. For example:

```
bm072 bf -pz 30
```

This finishes in seconds on any modern hardware. The console will
print a `TREASURE KEY FOUND!` line, the program will stop, and a
file named `FOUND_PUZZLE_30.txt` will appear in the current working
directory. Open that file to see the result format end-to-end. That
is exactly what you will get when you recover one of your own
wallets or solve a low-difficulty puzzle. For higher puzzles you
will see a similar but not identical file, which also includes
instructions for contacting the author.

## Updating the program (`bmXXX upgrade`)

The program can download a fresh release of itself. Run (with `./` on
Linux/macOS, without on Windows):

```
bmXXX upgrade
```

It will contact the official download server, check whether a newer
release is available for the same platform variant as your current
binary, and if so download the matching ZIP and extract the new
executable into the **current working directory**.

A few things to keep in mind:

- The new binary has a different file name — `bmYYY` on Linux/macOS or
  `bmYYY.exe` on Windows, where `YYY` is the new version number. Your
  old `bmXXX` file is **not** deleted; you can remove it manually
  once you are sure the new version works.
- After the upgrade, run the program using the new file name (e.g.
  `./bm073 dig -s ...` instead of `./bm072 dig -s ...`).
- Map and state files (`bf_*.map`, `bf_*.{cpu,cuda_N,amdgpu_N}.state`)
  are kept and are picked up by the new version, so your bruteforce
  progress is preserved.

If you do not follow the Telegram channel or the user group, running
`upgrade` from time to time is a simple way to stay on a recent
version.

## Contacts

- **Telegram channel** (announcements, releases, news):
  [https://t.me/BtcMole](https://t.me/BtcMole)
- **Telegram user group** (questions, setup help, sharing experience):
  [https://t.me/+AIPjS1v-u3FlZjZi](https://t.me/+AIPjS1v-u3FlZjZi)
- **Support** (registration, sessions, payouts, anything that does
  not work as expected):
  [@BtcMoleSup](https://t.me/BtcMoleSup)
- **Author**: [@keymole](https://t.me/keymole)

## License

See [LICENSE](LICENSE).
