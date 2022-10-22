# Maintainer: Philip Müller <philm[at]manjaro[dog]org>

pkgname=calamares
pkgver=3.2.61
_pkgver=3.2.61
pkgrel=6
_commit=8a6b3d19e17d5a92d0ad8f743c55965f03ff9fac
pkgdesc='Distribution-independent installer framework'
arch=('i686' 'x86_64')
license=(GPL)
url="https://gitlab.manjaro.org/applications/calamares"
license=('LGPL')
depends=('kconfig' 'kcoreaddons' 'kiconthemes' 'ki18n' 'kio' 'solid' 'yaml-cpp' 'kpmcore>=22.04.0' 'mkinitcpio-openswap'
         'boost-libs' 'ckbcomp' 'hwinfo' 'qt5-svg' 'polkit-qt5' 'gtk-update-icon-cache' 'plasma-framework'
         'qt5-xmlpatterns' 'squashfs-tools' 'libpwquality' 'appstream-qt' 'icu' 'python' 'qt5-webview'
         'ttf-comfortaa')
makedepends=('extra-cmake-modules' 'qt5-tools' 'qt5-translations' 'git' 'boost' 'kparts' 'kdbusaddons')
backup=('usr/share/calamares/modules/bootloader.conf'
        'usr/share/calamares/modules/displaymanager.conf'
        'usr/share/calamares/modules/initcpio.conf'
        'usr/share/calamares/modules/unpackfs.conf')

source+=(#"$pkgname-$pkgver-$pkgrel.tar.gz::$url/-/archive/v$pkgver/calamares-v$pkgver.tar.gz"
         'https://gitlab.manjaro.org/codesardine/calamares/-/commit/b140b67c9fddb96701e46d23e9a72ddfbe77e0d0.patch'
         'https://gitlab.manjaro.org/applications/calamares/-/merge_requests/37.patch'
         "$pkgname-$pkgver-$pkgrel.tar.gz::$url/-/archive/$_commit/$pkgname-$_commit.tar.gz"
        )
sha256sums=('39c38180b6c7d6088984c300e3fdf125b571525d1d252b59a52388e1780f98e4'
            '993c4d3babd7ead9a20fb43c338d3223f1531c55a2bdb0bf2dd899db22433214'
            '081415e5d6a44691cb984ccaea276e113ec8a058ce12dd3b774640ad44aeacd8')

prepare() {
	mv ${srcdir}/calamares-${_commit} ${srcdir}/calamares-${pkgver}
	#mv ${srcdir}/calamares-v${pkgver} ${srcdir}/calamares-${pkgver}
	cd ${srcdir}/calamares-${pkgver}
	sed -i -e 's/"Install configuration files" OFF/"Install configuration files" ON/' CMakeLists.txt
	
	# change version
	sed -i -e "s|$pkgver|$_pkgver|g" CMakeLists.txt
	#_ver="$(cat CMakeLists.txt | grep -m3 -e "  VERSION" | grep -o "[[:digit:]]*" | xargs | sed s'/ /./g')"
	_ver="$pkgver"
	printf 'Version: %s-%s' "${_ver}" "${pkgrel}"
	echo ""
	sed -i -e "s|\${CALAMARES_VERSION_MAJOR}.\${CALAMARES_VERSION_MINOR}.\${CALAMARES_VERSION_PATCH}|${_ver}-${pkgrel}|g" CMakeLists.txt
	sed -i -e "s|CALAMARES_VERSION_RC 1|CALAMARES_VERSION_RC 0|g" CMakeLists.txt
	
	# https://github.com/calamares/calamares/issues/2014
	sed -i -e 's|"$@"|"-D6" "$@"|g' data/calamares_polkit

	# change branding
	sed -i -e "s/default/manjaro/g" src/branding/CMakeLists.txt
	
	# https://github.com/calamares/calamares/issues/1945
	patch -Np1 -i ../b140b67c9fddb96701e46d23e9a72ddfbe77e0d0.patch
	
	# https://gitlab.manjaro.org/applications/calamares/-/merge_requests/37
	patch -Np1 -i ../37.patch
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
              -DBoost_NO_BOOST_CMAKE=ON \
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
