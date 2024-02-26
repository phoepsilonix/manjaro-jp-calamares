# Maintainer: Philip Müller <philm[at]manjaro[dog]org>

pkgname=calamares
pkgver=3.3.2
_pkgver=3.3.2
pkgrel=0
_commit=43eb5c0d4afba55b2b730ad0f8a14a01215e8c9c
pkgdesc='Distribution-independent installer framework'
arch=('i686' 'x86_64')
license=('BSD-2-Clause AND CC0-1.0 AND CC-BY-4.0 AND GPL-3.0-or-later AND LGPL-2.0-only AND LGPL-2.1-only AND LGPL-3.0-or-later AND MIT')
url="https://gitlab.manjaro.org/applications/calamares"
depends=('kconfig5' 'kcoreaddons5' 'kiconthemes5' 'ki18n5' 'kio5' 'solid5' 'yaml-cpp' 'kpmcore>=22.04.0' 'mkinitcpio-openswap'
         'ckbcomp' 'hwinfo' 'qt5-svg' 'polkit-qt5' 'gtk-update-icon-cache' 'plasma-framework5'
         'qt5-xmlpatterns' 'squashfs-tools' 'libpwquality' 'appstream-qt5' 'icu' 'python' 'qt5-webview')
makedepends=('extra-cmake-modules' 'qt5-tools' 'qt5-translations' 'git' 'kparts5' 'kdbusaddons5'
             'qt5-webengine')
optdepends=('kparts5: for interactiveterminal module'
            'plasma-framework5: for plasmalnf module'
            'qt5-webengine: for webview module')
backup=('usr/share/calamares/modules/bootloader.conf'
        'usr/share/calamares/modules/displaymanager.conf'
        'usr/share/calamares/modules/initcpio.conf'
        'usr/share/calamares/modules/unpackfs.conf')

source+=(#"$pkgname-$pkgver.tar.gz::$url/-/archive/v$pkgver/calamares-v$pkgver.tar.gz"
         "$pkgname-$pkgver-$pkgrel.tar.gz::$url/-/archive/$_commit/$pkgname-$_commit.tar.gz"
        )
sha256sums=('f3f5201124b04a88a57aa40fda3bcdc682c9cd40f057a0ad7895fc79fa444797')

prepare() {
	mv ${srcdir}/calamares-${_commit} ${srcdir}/calamares-${pkgver}
	#mv ${srcdir}/calamares-v${pkgver} ${srcdir}/calamares-${pkgver}
	cd ${srcdir}/calamares-${pkgver}
	
	# change version
	sed -i -e "s|$pkgver|$_pkgver|g" CMakeLists.txt
	#_ver="$(cat CMakeLists.txt | grep -m3 -e "  VERSION" | grep -o "[[:digit:]]*" | xargs | sed s'/ /./g')"
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

	mkdir -p build
	cd build
        cmake .. \
              -DCMAKE_BUILD_TYPE=Debug \
              -DCMAKE_INSTALL_PREFIX=/usr \
              -DCMAKE_INSTALL_LIBDIR=lib \
              -DWITH_KF5DBus=OFF \
              -DINSTALL_CONFIG=ON \
              -DSKIP_MODULES="initramfs initramfscfg \
                              dummyprocess dummypython \
                              dummycpp dummypythonqt \
                              services-openrc"
        make
}

package() {
	cd ${srcdir}/calamares-${pkgver}/build
	make DESTDIR="$pkgdir" install
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
	
	# fix branding install
	cp -av "../src/branding/manjaro" "$pkgdir/usr/share/calamares/branding/"
}
