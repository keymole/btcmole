# BtcMole

BtcMole is a tool for solving Bitcoin puzzles (#71 and higher) and for
recovering forgotten private keys when you know the key range and your
own P2PKH address.

The program runs on CPU and on NVIDIA / AMD GPUs (Linux, Windows,
macOS arm64, Linux arm/arm64). Pre-built binaries for every supported
platform are published in this repository as ZIP archives.

### Which build to download

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
`bm072.linux_amd64_cuda.zip`; on a Linux box with an AMD card take
`bm072.linux_amd64_amdgpu.zip`; on a Linux box with both — take
`bm072.linux_amd64_gpu.zip`.

The CPU-only build (`bm072.linux_amd64.zip` / `bm072.windows_amd64.zip`)
is the smallest and has no GPU runtime requirements at all — useful on
servers / VPS with no GPU. The GPU builds also work fine without a
GPU (they just fall back to CPU), but they are larger because they
embed precompiled GPU kernels.

For non-amd64 platforms (Linux arm, Linux arm64, macOS arm64) only
the CPU build is published — there is no `_cuda` / `_amdgpu` / `_gpu`
variant for those.

### A note on the executable name

Every release of the program ships with the version number baked into
the executable file name: `bmXXX`, where `XXX` is the three-digit
version. For example, the current release `bm072.linux_amd64.zip`
unpacks to an executable called `bm072`. The next release will be
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

### Android

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

## License

See [LICENSE](LICENSE).
