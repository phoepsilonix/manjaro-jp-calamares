# Maintainer: Philip Müller <philm[at]manjaro[dog]org>

pkgname=calamares
pkgver=3.4.0
_pkgver=3.4.0
pkgrel=6
_commit=037a317fa986034d02cc37e84ddcf3c2119565df
pkgdesc='Distribution-independent installer framework'
arch=('i686' 'x86_64')
url="https://gitlab.manjaro.org/applications/calamares"
depends=('kconfig' 'kcoreaddons' 'kiconthemes' 'ki18n' 'solid' 'yaml-cpp' 'kpmcore'
	'boost-libs' 'ckbcomp' 'hwinfo' 'qt6-svg' 'polkit-qt6'
	'squashfs-tools' 'libpwquality' 'python')
backup=('usr/share/calamares/modules/bootloader.conf'
        'usr/share/calamares/modules/displaymanager.conf'
        'usr/share/calamares/modules/initcpio.conf'
        'usr/share/calamares/modules/unpackfs.conf')
license=(
  'BSD-2-Clause'
  'CC0-1.0'
  'CC-BY-4.0'
  'GPL-3.0-or-later'
  'LGPL-2.0-only'
  'LGPL-2.1-only'
  'LGPL-3.0-or-later'
  'MIT'
)
depends=(
  'boost-libs'
  'ckbcomp'
  'hwinfo'
  'kconfig'
  'kcoreaddons'
  'kcrash'
  'kiconthemes'
  'ki18n'
  'kpmcore'
  'libpwquality'
  'polkit-qt6'
  'python'
  'qt6-svg'
  'solid'
  'squashfs-tools'
  'yaml-cpp'
)
makedepends=(
  'boost'
  'cmake'
  'extra-cmake-modules'
  'git'
  'qt6-tools'
  'qt6-translations'
)
backup=(
  'usr/share/calamares/modules/bootloader.conf'
  'usr/share/calamares/modules/displaymanager.conf'
  'usr/share/calamares/modules/initcpio.conf'
  'usr/share/calamares/modules/unpackfs.conf'
)
source=(
#  "$pkgname-$pkgver.tar.gz::$url/-/archive/v$pkgver/calamares-v$pkgver.tar.gz"
  "$pkgname-$pkgver-$pkgrel.tar.gz::$url/-/archive/$_commit/$pkgname-$_commit.tar.gz"
  "001-no-oom.patch"
  "manjaro_jp.patch"
)
sha256sums=('edb09ad7dca23fe329d10b86c097aa381f000d7541ec7e4ecfa2f4be837b2a97'
            '57d905dd62e320938b3288f8713762b7acca68deb6b35be4916bc7031a706f1a'
            '9c5374a60b73de0f26e1b376e99e01ab357c9a65d4e6472da64b977e80cfe58b')

prepare() {
  mv ${srcdir}/calamares-${_commit} ${srcdir}/calamares-${pkgver}
#  mv ${srcdir}/calamares-v${pkgver} ${srcdir}/calamares-${pkgver}
  cd ${srcdir}/calamares-${pkgver}
  
  # change version
  sed -i -e "s|$pkgver|$_pkgver|g" CMakeLists.txt
#  _ver="$(cat CMakeLists.txt | grep -m3 -e "  VERSION" | grep -o "[[:digit:]]*" | xargs | sed s'/ /./g')"
  _ver="$pkgver"
  printf 'Version: %s-%s' "${_ver}" "${pkgrel}"
  echo ""
  sed -i -e "s|\${CALAMARES_VERSION_MAJOR}.\${CALAMARES_VERSION_MINOR}.\${CALAMARES_VERSION_PATCH}|${_ver}-${pkgrel}|g" CMakeLists.txt
  sed -i -e "s|CALAMARES_VERSION_RC 1|CALAMARES_VERSION_RC 0|g" CMakeLists.txt

  # change branding
  sed -i -e "s/default/manjaro/g" src/branding/CMakeLists.txt
  
  # Apply patches
  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    [[ $src = *.patch ]] || continue
    msg2 "Applying patch: $src..."
    patch -Np1 < "../$src"
  done
}

build() {
  cd ${srcdir}/calamares-${pkgver}

    export CFLAGS="$CFLAGS -O0"
    export CXXFLAGS="$CXXFLAGS -O0"
	mkdir -p build
	cd build
        cmake .. \
              -DCMAKE_BUILD_TYPE=Debug \
              -DCMAKE_INSTALL_PREFIX=/usr \
              -DCMAKE_INSTALL_LIBDIR=lib \
              -DWITH_QT6=ON \
              -DINSTALL_CONFIG=ON \
              -DCMAKE_SHARED_LINKER_FLAGS="$LDFLAGS" \
              -DSKIP_MODULES="initramfs initramfscfg \
                              dummyprocess dummypython \
                              dummycpp dummypythonqt \
                              services-openrc"
        cmake --build .
}

package() {
	cd ${srcdir}/calamares-${pkgver}/build
	DESTDIR="$pkgdir" cmake --install .
	install -Dm644 "../data/manjaro-icon.svg" "$pkgdir/usr/share/icons/hicolor/scalable/apps/calamares.svg"
	install -Dm644 "../data/calamares.desktop" "$pkgdir/usr/share/applications/calamares.desktop"
	install -Dm755 "../data/calamares_polkit" "$pkgdir/usr/bin/calamares_polkit"
	install -Dm644 "../data/49-nopasswd-calamares.rules" "$pkgdir/etc/polkit-1/rules.d/49-nopasswd-calamares.rules"
	chmod 750      "$pkgdir"/etc/polkit-1/rules.d

	# rename services-systemd back to services
	mv "$pkgdir/usr/lib/calamares/modules/services-systemd" "$pkgdir/usr/lib/calamares/modules/services"
	mv "$pkgdir/usr/share/calamares/modules/services-systemd.conf" "$pkgdir/usr/share/calamares/modules/services.conf"
	sed -i -e 's/-systemd//' "$pkgdir/usr/lib/calamares/modules/services/module.desc"
	sed -i -e 's/-systemd//' "$pkgdir/usr/share/calamares/settings.conf"
	
    # Added LICENSES files
    mkdir -p "$pkgdir/usr/share/licenses/calamares/"
    for L in BSD-2-Clause CC-BY-4.0 CC0-1.0 GPL-3.0-or-later LGPL-2.1-only LGPL-3.0-or-later MIT;
    do
	    install -Dm644 "../LICENSES/$L.txt" "$pkgdir/usr/share/licenses/calamares/"
    done
	# fix branding install
	cp -av "../src/branding/manjaro" "$pkgdir/usr/share/calamares/branding/"
}
