# Maintainer: Philip Müller <philm[at]manjaro[dog]org>

pkgname=calamares
pkgver=3.2.1
_pkgver=3.2.1
pkgrel=5
pkgdesc='Distribution-independent installer framework'
arch=('i686' 'x86_64')
license=(GPL)
url="https://gitlab.manjaro.org/applications/calamares"
license=('LGPL')
depends=('kconfig' 'kcoreaddons' 'kiconthemes' 'ki18n' 'kio' 'solid' 'yaml-cpp' 'kpmcore>=3.3.0' 'mkinitcpio-openswap'
         'boost-libs' 'ckbcomp' 'hwinfo' 'qt5-svg' 'polkit-qt5' 'gtk-update-icon-cache' 'pythonqt>=3.2')
makedepends=('extra-cmake-modules' 'qt5-tools' 'qt5-translations' 'git' 'boost')
backup=('usr/share/calamares/modules/bootloader.conf'
        'usr/share/calamares/modules/displaymanager.conf'
        'usr/share/calamares/modules/initcpio.conf'
        'usr/share/calamares/modules/unpackfs.conf')

source+=(#"$pkgname-$pkgver.tar.gz::https://github.com/manjaro/calamares/archive/v${_pkgver}.tar.gz"
         "$pkgname-$pkgver-$pkgrel.tar.gz::$url/-/archive/3.2.x-stable/calamares-3.2.x-stable.tar.gz"
         https://github.com/calamares/calamares/pull/1020.patch
         https://github.com/calamares/calamares/pull/1021.patch
         https://github.com/calamares/calamares/pull/1022.patch)
sha256sums=('26c6103c221ba7896fae251d59a52e8451fa8a3e9f25975c31a2a8c95f9f84e0'
            'bbbd04123b42fc63308398227c9380607997f3998b1734f90d208ff17152e340'
            '868a90fde6264f38cd3ec36ca585ff145b59a648bf634c086c93249d3605b30c'
            '05f501f745688384dcd5f72079e892b5331fe30ebd931fe2c9e337f040ab696a')

prepare() {
	mv ${srcdir}/calamares-3.2.x-stable ${srcdir}/calamares-${_pkgver}
	cd ${srcdir}/calamares-${_pkgver}

	# patches here
	patch -p1 -i ../1020.patch
	patch -p1 -i ../1021.patch
	patch -p1 -i ../1022.patch

	# add revision
	_patchver="$(cat CMakeLists.txt | grep -m3 -e CALAMARES_VERSION_PATCH | grep -o "[[:digit:]]*" | xargs)"
	sed -i -e "s|CALAMARES_VERSION_PATCH $_patchver|CALAMARES_VERSION_PATCH $_patchver-$pkgrel|g" CMakeLists.txt
}

build() {
	cd ${srcdir}/calamares-${_pkgver}

	mkdir -p build
	cd build
        cmake .. \
              -DCMAKE_BUILD_TYPE=Release \
              -DCMAKE_INSTALL_PREFIX=/usr \
              -DCMAKE_INSTALL_LIBDIR=lib \
              -DWITH_PYTHONQT:BOOL=ON \
              -DSKIP_MODULES="webview interactiveterminal initramfs \
                              initramfscfg dracut dracutlukscfg \
                              dummyprocess dummypython dummycpp \
                              dummypythonqt plasmalnf"
        make
}

package() {
	cd ${srcdir}/calamares-${_pkgver}/build
	make DESTDIR="$pkgdir" install
	install -Dm644 "../data/manjaro-icon.svg" "$pkgdir/usr/share/icons/hicolor/scalable/apps/calamares.svg"
	install -Dm644 "../data/calamares.desktop" "$pkgdir/usr/share/applications/calamares.desktop"
	install -Dm755 "../data/calamares_polkit" "$pkgdir/usr/bin/calamares_polkit"
	install -Dm644 "../data/49-nopasswd-calamares.rules" "$pkgdir/etc/polkit-1/rules.d/49-nopasswd-calamares.rules"
	chmod 750      "$pkgdir"/etc/polkit-1/rules.d
}
