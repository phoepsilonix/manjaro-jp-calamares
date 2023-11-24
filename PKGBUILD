# Maintainer: Philip Müller <philm[at]manjaro[dog]org>

pkgname=calamares
pkgver=3.2.62
_pkgver=3.2.62
pkgrel=10
_commit=8a91c36ad1491939d90c648853180ba892060f3a
pkgdesc='Distribution-independent installer framework'
arch=('i686' 'x86_64')
license=(GPL)
url="https://gitlab.manjaro.org/applications/calamares"
license=('LGPL')
depends=('kconfig5' 'kcoreaddons5' 'kiconthemes5' 'ki18n5' 'kio5' 'solid5' 'yaml-cpp' 'kpmcore>=22.04.0' 'mkinitcpio-openswap'
         'boost-libs' 'ckbcomp' 'hwinfo' 'qt5-svg' 'polkit-qt5' 'gtk-update-icon-cache' 'plasma-framework5'
         'qt5-xmlpatterns' 'squashfs-tools' 'libpwquality' 'appstream-qt5' 'icu' 'python' 'qt5-webview'
         'ttf-comfortaa')
makedepends=('extra-cmake-modules' 'qt5-tools' 'qt5-translations' 'git' 'boost' 'kparts5' 'kdbusaddons5')
backup=('usr/share/calamares/modules/bootloader.conf'
        'usr/share/calamares/modules/displaymanager.conf'
        'usr/share/calamares/modules/initcpio.conf'
        'usr/share/calamares/modules/unpackfs.conf')

source+=("$pkgname-$pkgver-$pkgrel.tar.gz::$url/-/archive/v$pkgver/calamares-v$pkgver.tar.gz"
         'https://gitlab.manjaro.org/applications/calamares/-/commit/8c873e0f49cef09a83a26c4ffc073925e1a91d4d.patch'
         'a6dd49ac0789ae172b2e00b04a665a3dfce09590.patch'
         '757f8a8f9ed2c226bc1064d54de1fe3fe7ba3974.patch'
         'arch-appstream-qt5.patch'
         #"$pkgname-$pkgver-$pkgrel.tar.gz::$url/-/archive/$_commit/$pkgname-$_commit.tar.gz"
        )
sha256sums=('d9ecc6e5757ba3dcf2f3c3fa68c67508cdffed665c7c0d8895bcb0a5e9fbbbfd'
            '66e78ec6e9ea0152ba8862d49afd74bf9cd64bd6aa3ef8173b851c122a241f45'
            'f76965f4729c5b707862c4c352e3603d719872a5add5f7bd822ede878404e938'
            'a416e6205faf215345c6b121dc05a72f76b5c028e20085159e2c80132183d78d'
            '585f10bb1b15e9a57d71b3da9cd969ca683f0fb81321ee8cc24371fad33fe247')

prepare() {
	#mv ${srcdir}/calamares-${_commit} ${srcdir}/calamares-${pkgver}
	mv ${srcdir}/calamares-v${pkgver} ${srcdir}/calamares-${pkgver}
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
	patch -Np1 -i ../8c873e0f49cef09a83a26c4ffc073925e1a91d4d.patch
	
	# Fix Appstream 1.0
	patch -Np1 -i ../a6dd49ac0789ae172b2e00b04a665a3dfce09590.patch
        patch -Np1 -i ../757f8a8f9ed2c226bc1064d54de1fe3fe7ba3974.patch
        echo "done"
        patch -Np1 -i ../arch-appstream-qt5.patch
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
