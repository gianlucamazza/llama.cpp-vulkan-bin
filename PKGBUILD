# Maintainer: Gianluca Mazza <info@gianlucamazza.it>
pkgname=llama.cpp-vulkan-bin
pkgver=b9518
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
sha256sums=('bc5fd4391dd546d6fb088c4347241444e2bebd83e52d10040ddf23d0ae494e07')

package() {
    cd "${srcdir}/llama-${pkgver}"

    install -dm755 \
        "${pkgdir}/usr/bin" \
        "${pkgdir}/usr/lib" \
        "${pkgdir}/usr/share/licenses/${pkgname}"

    # binaries
    for f in llama-* llama rpc-server; do
        [[ -f "$f" && -x "$f" && "$f" != *.so* ]] && install -Dm755 "$f" "${pkgdir}/usr/bin/$f"
    done

    # shared libraries — install full-versioned files first, then symlink the others
    for f in *.so *.so.*; do
        [[ -f "$f" ]] && install -Dm644 "$f" "${pkgdir}/usr/lib/$f"
    done

    install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"

    # replace duplicate versioned copies with proper symlinks (libfoo.so.X, libfoo.so)
    cd "${pkgdir}/usr/lib"
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
