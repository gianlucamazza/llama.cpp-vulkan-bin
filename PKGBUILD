# Maintainer: Gianluca Mazza <info@gianlucamazza.it>
pkgname=llama.cpp-vulkan-bin
pkgver=b10443
pkgrel=1
pkgdesc="llama.cpp prebuilt Vulkan binaries for Linux x86_64"
arch=('x86_64')
url='https://github.com/ggml-org/llama.cpp'
license=('MIT')
provides=('llama.cpp')
conflicts=('llama.cpp' 'llama.cpp-bin' 'libggml' 'ggml')
depends=('curl' 'gcc-libs' 'glibc' 'libstdc++' 'openssl' 'vulkan-icd-loader')
optdepends=(
    'vulkan-intel: Intel GPU acceleration'
    'vulkan-radeon: AMD GPU acceleration'
    'python-numpy: convert_hf_to_gguf.py'
    'python-gguf: convert_hf_to_gguf.py'
)
source=("llama-${pkgver}-bin-ubuntu-vulkan-x64.tar.gz::https://github.com/ggml-org/llama.cpp/releases/download/${pkgver}/llama-${pkgver}-bin-ubuntu-vulkan-x64.tar.gz")
sha256sums=('d5189031d7370fb7f1c1470ae4d57a8f2d6459f82d9c09235ab2b17ccfa8832e')

# Layout: libexec-style. ggml's runtime backend loader (plugins libggml-cpu-<variant>.so,
# libggml-vulkan.so) searches ONLY the executable's directory and the CWD — never the
# system libdir — and GGML_BACKEND_PATH cannot drive CPU-variant selection (it names a
# single file). Splitting binaries (/usr/bin) from plugins (/usr/lib) therefore breaks
# every launcher with "no backends are loaded" unless started from /usr/lib.
# Fix at the package layer: real binaries AND all libs live together in /usr/lib/llama.cpp/
# (binaries have RUNPATH=$ORIGIN, verified), /usr/bin holds symlinks — /proc/self/exe
# resolves them, so the loader sees the real dir and variant-selects correctly from any CWD.
package() {
    cd "${srcdir}/llama-${pkgver}"

    local appdir="usr/lib/llama.cpp"
    install -dm755 \
        "${pkgdir}/usr/bin" \
        "${pkgdir}/${appdir}" \
        "${pkgdir}/usr/share/licenses/${pkgname}"

    # binaries → appdir, launcher symlinks → /usr/bin
    for f in llama-* llama rpc-server; do
        if [[ -f "$f" && -x "$f" && "$f" != *.so* ]]; then
            install -Dm755 "$f" "${pkgdir}/${appdir}/$f"
            ln -s "/${appdir}/$f" "${pkgdir}/usr/bin/$f"
        fi
    done

    # shared libraries (incl. ggml backend plugins) → appdir, beside the binaries
    for f in *.so *.so.*; do
        [[ -f "$f" ]] && install -Dm644 "$f" "${pkgdir}/${appdir}/$f"
    done

    install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"

    # replace duplicate versioned copies with proper symlinks (libfoo.so.X, libfoo.so)
    cd "${pkgdir}/${appdir}"
    shopt -s nullglob
    for full in *.so.[0-9]*.[0-9]*.[0-9]*; do
        base="${full%%.so.*}"
        rest="${full#*.so.}"
        major="${rest%%.*}"
        rm -f "${base}.so.${major}" "${base}.so"
        ln -s "$full" "${base}.so.${major}"
        ln -s "${base}.so.${major}" "${base}.so"
    done
    shopt -u nullglob
}
