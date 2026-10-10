# Maintaining

This repository is the PKGBUILD source. CI and the bump workflow validate a
new `pkgver`; they do not publish a package. Built packages go to the personal
`gianluca` pacman repo (not AUR, and not GitHub Releases).

## Automated bump

`.github/workflows/bump.yml` runs every 6 hours (and on `workflow_dispatch`).
It checks the latest [llama.cpp](https://github.com/ggml-org/llama.cpp) release,
confirms the Ubuntu Vulkan x86_64 tarball exists, updates `pkgver`/`sha256` and
`.SRCINFO`, builds and smoke-tests the package (including a backend-loading
check from an arbitrary cwd), opens a PR, and auto-merges on success.

## Manual bump

If the scheduled job is skipped or you need an off-cycle bump:

1. Set `pkgver` in `PKGBUILD` to the upstream tag (for example `b10549`).
2. Run `updpkgsums` to refresh `sha256sums`.
3. Regenerate `.SRCINFO`: `makepkg --printsrcinfo > .SRCINFO`.
4. Push. CI will lint, rebuild, and smoke-test.

## Publish to the personal pacman repo

After a bump lands on `main`, build, sign, and publish from the personal
packaging tree that contains `publish.sh`:

```
cd "${ARCH_PACKAGES_DIR}/pkgs/llama.cpp-vulkan-bin" && git checkout main && git pull
cd "${ARCH_PACKAGES_DIR}" && ./publish.sh llama.cpp-vulkan-bin
sudo pacman -Syu llama.cpp-vulkan-bin
```

`ARCH_PACKAGES_DIR` is the checkout of that packaging tree. `publish.sh` builds
in a clean chroot, signs the package, updates the `gianluca` repo database, and
syncs it to the repo host (`REPO_HOST`, typically over rsync).
