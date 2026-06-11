# llama.cpp-vulkan-bin

Arch Linux package providing prebuilt **Vulkan** binaries of [llama.cpp](https://github.com/ggml-org/llama.cpp) for Linux x86_64.

Packages the official upstream release binaries — no local compilation required.

> Personal package, distributed from my own pacman repo — **not** the AUR. An unrelated
> `llama.cpp-vulkan-bin` exists on the AUR under a different maintainer; this is not it.

## Installation

Build straight from this repo (no extra setup needed):

```
git clone https://github.com/gianlucamazza/llama.cpp-vulkan-bin.git
cd llama.cpp-vulkan-bin
makepkg -si
```

Or, if you have the `gianluca` pacman repo configured:

```
sudo pacman -Syu llama.cpp-vulkan-bin
```

## What's included

- `llama-server` — OpenAI-compatible HTTP inference server
- `llama-cli` — interactive CLI chat
- `llama-bench`, `llama-quantize`, `llama-imatrix`, and all other upstream tools
- Vulkan backend shared libraries (`libggml-vulkan.so`, `libggml.so`, `libllama.so`, ...)

## Package layout (since b9592-1)

Everything is installed under `/usr/lib/llama.cpp/` (binaries and libraries together);
`/usr/bin` holds symlinks to the binaries.

This is deliberate, not cosmetic: ggml's runtime backend loader searches for its backend
plugins (`libggml-cpu-<variant>.so`, `libggml-vulkan.so`) **only in the executable's
directory and the current working directory** — never in the system libdir — and
`GGML_BACKEND_PATH` cannot drive CPU-variant selection (it points to a single file).
With the previous split layout (binaries in `/usr/bin`, libs in `/usr/lib`) every tool
aborted with `no backends are loaded` unless launched with `cwd=/usr/lib`. The upstream
binaries ship `RUNPATH=$ORIGIN` and `/proc/self/exe` resolves symlinks, so the co-located
layout makes backend loading work from any working directory.

## GPU acceleration

Vulkan is used automatically when a compatible GPU driver is present:

```
vulkan-intel   # Intel (Iris Xe, Arc)
vulkan-radeon  # AMD
```

Check detected devices:

```
llama-cli --list-devices
```

## Updating

A scheduled GitHub Action (`.github/workflows/bump.yml`) checks for new upstream releases
every 6h, opens a PR that bumps `pkgver`/`sha256`, builds + smoke-tests it (including a
backend-loading check from an arbitrary cwd), and auto-merges on success. To publish the
merged bump to the `gianluca` pacman repo:

```
cd ~/Workspace/tooling/arch-packages/pkgs/llama.cpp-vulkan-bin && git pull
cd ~/Workspace/tooling/arch-packages && ./publish.sh llama.cpp-vulkan-bin
sudo pacman -Syu llama.cpp-vulkan-bin
```

To bump by hand instead: edit `pkgver` in `PKGBUILD`, run `updpkgsums`, regenerate
`.SRCINFO` (`makepkg --printsrcinfo > .SRCINFO`) and push.

## License

PKGBUILD © 2026 Gianluca Mazza — MIT  
llama.cpp © The ggml authors — MIT
