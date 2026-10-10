# llama.cpp-vulkan-bin

Arch Linux package of the official [llama.cpp](https://github.com/ggml-org/llama.cpp) **Vulkan** prebuilt binaries for Linux x86_64. The PKGBUILD downloads `llama-<ver>-bin-ubuntu-vulkan-x64.tar.gz` from [upstream GitHub Releases](https://github.com/ggml-org/llama.cpp/releases) and installs it — no local compilation, and this repository does not publish GitHub Release assets of its own.

This is **not** the [AUR package of the same name](https://aur.archlinux.org/packages/llama.cpp-vulkan-bin). That listing is a different package under a different maintainer.

Built packages are signed and published to a personal pacman repo named `gianluca` (not AUR). If that repo is already configured, install with `pacman`. Otherwise clone this repository and build with `makepkg` — that is the path that works without extra setup.

## Installation

Clone this repo and build the package (no extra setup):

```
git clone https://github.com/gianlucamazza/llama.cpp-vulkan-bin.git
cd llama.cpp-vulkan-bin
makepkg -si
```

Or, if the personal `gianluca` pacman repo is already configured on the machine:

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

A scheduled workflow (`.github/workflows/bump.yml`) tracks new upstream releases.
Maintainer bump and publish steps live in [MAINTAINING.md](MAINTAINING.md).

## License

PKGBUILD © 2026 Gianluca Mazza — MIT  
llama.cpp © The ggml authors — MIT
