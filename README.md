# llama.cpp-vulkan-bin

Arch Linux package (AUR) providing prebuilt **Vulkan** binaries of [llama.cpp](https://github.com/ggml-org/llama.cpp) for Linux x86_64.

Packages the official upstream release binaries — no local compilation required.

## Installation

```
yay -S llama.cpp-vulkan-bin
```

Or via the AUR directly:

```
git clone https://aur.archlinux.org/llama.cpp-vulkan-bin.git
cd llama.cpp-vulkan-bin
makepkg -si
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

When a new upstream release is out, bump `pkgver` in `PKGBUILD`, run `updpkgsums`, regenerate `.SRCINFO` and push.

## License

PKGBUILD © 2026 Gianluca Mazza — MIT  
llama.cpp © The ggml authors — MIT
