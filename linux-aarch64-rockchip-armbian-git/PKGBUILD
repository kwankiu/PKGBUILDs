# AArch64 multi-platform
# Maintainer: Jat-faan Wong
# Contributor: Jat-faan Wong 

pkgbase=linux-aarch64-rockchip-armbian-git
pkgname=("${pkgbase}"{,-headers})
pkgver=6.10.0.rc6.r18.a422ac.088d3b
pkgrel=1
arch=('aarch64')
license=('GPL2')
url="https://kernel.org"
_desc="with patches picked from armbian on RK3588" 
makedepends=('cpio' 'xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'uboot-tools' 'vboot-utils' 'dtc')
options=('!strip')
_srcname=linux-stable
_buildname='build'

source=(
  "${_srcname}::git+https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git#branch=master"
  "git+https://github.com/armbian/${_buildname}.git"
  'custom_reconfig'
)

sha512sums=(
  'SKIP'
  'SKIP'
  '5491e2c4d69b37a8fc8761e128cef89b3406754850c71486e86f7bdba5daf855574e191b2f4d75c654a80eff33ea5249efc9ad6a97d9805cee9e5a80ed93c302'
)

_config=config/kernel/linux-rockchip-rk3588-6.10.config
_patch=patch/kernel/archive/rockchip-rk3588-6.10

prepare() {

  echo "Setting version..."
  cd ${_buildname}
  local rev_config="$(git rev-list --count HEAD "${_config}")"
  local id_config="$(git rev-parse --short=6 HEAD:"${_config}")"

  local rev_patch="$(git rev-list --count HEAD "${_patch}")"
  local id_patch="$(git rev-parse --short=6 HEAD:"${_patch}")"

  cd ../${_srcname}

  echo . > localversion.09-hyphen
  echo "r$(( "${rev_config}" + "${rev_patch}" ))" > localversion.10-release-total
  echo . > localversion.19-hyphen
  echo "${id_config}" > localversion.20-id-config
  echo . > localversion.29-hyphen
  echo "${id_patch}" > localversion.30-id-patch
  echo "-${pkgrel}" > localversion.40-pkgrel

  # this is only for local builds so there is no need to integrity check. (if needed)
  for p in ../${_buildname}/${_patch}/*.patch; do
    echo "Custom Patching with ${p}"
    patch -p1 -N -i $p || true
  done

  # add custom dt and overlay
  echo "Copying custom dt ..."
  cp -rv ../${_buildname}/${_patch}/dt/*.dts arch/arm64/boot/dts/rockchip
  echo "Copying custom overlay ..."
  cp -rv ../${_buildname}/${_patch}/overlay arch/arm64/boot/dts/rockchip/overlay

  # append newly added dts to Makefile
  for p in ../${_buildname}/${_patch}/dt/*.dts; do
    echo "Adding $(basename $p) to Makefile"
    echo "dtb-\$(CONFIG_ARCH_ROCKCHIP) += $(basename $p)" >> arch/arm64/boot/dts/rockchip/Makefile
  done

  echo "Preparing config..."
  cat ../${_buildname}/${_config} > .config
  scripts/kconfig/merge_config.sh -m .config ../custom_reconfig
  make olddefconfig prepare

  make -s kernelrelease > version
  echo "Prepared for $(<version)"
}

pkgver() {
  cd "${_srcname}"
  local major_minor_patch=$(make kernelversion)
  local release_total=$(< localversion.10-release-total)
  local id_config=$(< localversion.20-id-config)
  local id_patch=$(< localversion.30-id-patch)

  # Remove unsupported characters from the version components
  major_minor_patch="${major_minor_patch//[-\/:\ ]/.}"
  release_total="${release_total//[-\/:\ ]/.}"
  id_config="${id_config//[-\/:\ ]/.}"
  id_patch="${id_patch//[-\/:\ ]/.}"

  # Construct the final version string
  printf '%s.%s.%s.%s' "$major_minor_patch" "$release_total" "$id_config" "$id_patch"
}

build() {
  cd "${_srcname}"

  unset LDFLAGS
  make ${MAKEFLAGS} Image modules
  make ${MAKEFLAGS} DTC_FLAGS="-@" dtbs
}

_package() {
  pkgdesc="The linux kernel, ${_desc}"
  depends=('coreutils' 'kmod' 'initramfs')
  optdepends=('wireless-regdb: to set the correct wireless channels of your country')

  cd "${_srcname}"
  
  # install dtbs
  make INSTALL_DTBS_PATH="${pkgdir}/boot/dtbs/${pkgbase}" dtbs_install

  # install modules
  make INSTALL_MOD_PATH="${pkgdir}/usr" INSTALL_MOD_STRIP=1 modules_install

  # copy kernel
  local _dir_module="${pkgdir}/usr/lib/modules/$(<version)"
  install -Dm644 arch/arm64/boot/Image "${_dir_module}/vmlinuz"

  # remove reference to build host
  rm -f "${_dir_module}/"{build,source}

  # used by mkinitcpio to name the kernel
  echo "${pkgbase}" | install -D -m 644 /dev/stdin "${_dir_module}/pkgbase"
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the Linux kernel, ${_desc}"
  depends=("python")

  cd "${_srcname}"
  local builddir="${pkgdir}/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map version
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/arm64" -m644 arch/arm64/Makefile
  cp -t "$builddir" -a scripts

  # add xfs and shmem for aufs building
  mkdir -p "$builddir"/{fs/xfs,mm}

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/arm64" -a arch/arm64/include
  install -Dt "$builddir/arch/arm64/kernel" -m644 arch/arm64/kernel/asm-offsets.s
  mkdir -p "$builddir/arch/arm"
  cp -t "$builddir/arch/arm" -a arch/arm/include

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */arm64/ || $arch == */arm/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  echo "Removing documentation..."
  rm -r "$builddir/Documentation"

  echo "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  echo "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -bi "$file")" in
      application/x-sharedlib\;*)      # Libraries (.so)
        strip -v $STRIP_SHARED "$file" ;;
      application/x-archive\;*)        # Libraries (.a)
        strip -v $STRIP_STATIC "$file" ;;
      application/x-executable\;*)     # Binaries
        strip -v $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*) # Relocatable binaries
        strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name -print0)

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#${pkgbase}}
  }"
done
