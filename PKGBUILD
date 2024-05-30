# Maintainer: Philip Müller <philm[at]manjaro[dog]org>

pkgname=calamares5
pkgver=3.2.62
_pkgver=3.2.62
pkgrel=9
_commit=6a667e4157a2107c7078c58a32e14079ded24a3c
pkgdesc='Distribution-independent installer framework'
arch=('i686' 'x86_64')
license=('BSD-2-Clause AND CC0-1.0 AND CC-BY-4.0 AND GPL-3.0-or-later AND LGPL-2.0-only AND LGPL-2.1-only AND LGPL-3.0-or-later AND MIT')
url="https://gitlab.manjaro.org/applications/calamares"
conflicts=('calamares')
provides=("calamares=$pkgver")
depends=('kconfig5' 'kcoreaddons5' 'ki18n5' 'kio5' 'solid5' 'yaml-cpp' 'kpmcore5>=23.05.0' 'mkinitcpio-openswap'
         'boost-libs' 'ckbcomp' 'hwinfo' 'qt5-svg' 'polkit-qt5' 'gtk-update-icon-cache' 'plasma-framework5'
         'qt5-xmlpatterns' 'squashfs-tools' 'libpwquality' 'appstream-qt5' 'icu' 'python' 'qt5-webview'
         'ttf-comfortaa'
kauth5
kwidgetsaddons5
kjobwidgets5
qt5-base
qt5-webengine
libx11
qt5-webchannel
parted
gcc-libs
qt5-declarative
qt5-location
kconfigwidgets5
hicolor-icon-theme
libxcrypt
kcompletion5
kxmlgui5
sonnet5
kpackage5
kwindowsystem5
bash
kservice5
ktextwidgets5
kcodecs5
glibc
kcrash5
kparts5
qt6-declarative
         )

makedepends=('extra-cmake-modules' 'qt5-tools' 'qt5-translations' 'git' 'boost' 'kparts5' 'kdbusaddons5')
backup=('usr/share/calamares/modules/bootloader.conf'
        'usr/share/calamares/modules/displaymanager.conf'
        'usr/share/calamares/modules/initcpio.conf'
        'usr/share/calamares/modules/unpackfs.conf')

source+=(#"calamares-$pkgver.tar.gz::$url/-/archive/v$pkgver/calamares-v$pkgver.tar.gz"
         'arch-appstream-qt5.patch'
         '2246-v32.patch'
         001-no-oom.patch
         manjaro_jp.patch
         "calamares-$pkgver-$pkgrel.tar.gz::$url/-/archive/$_commit/calamares-$_commit.tar.gz"
        )
sha256sums=('d46d58816f3713f5468a3f120c7613a23aa66d47a1b0c38c441f856056d7c993'
            '6044d672a896200fbd319795bfd40a1c012e4ef6cf0dafeeae7e1d021d92d96f'
            '57d905dd62e320938b3288f8713762b7acca68deb6b35be4916bc7031a706f1a'
            '53027b2c2a1d5a4180cc50c144787097e2a886c1820b631109b403b4debd1369'
            'a6bbab39ac9f6b791d9f10c40a34d46925b2078bbf9ec2f7f249e6c8d9f93998')

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
              -DBoost_NO_BOOST_CMAKE=ON \
              -DCMAKE_SHARED_LINKER_FLAGS="$LDFLAGS" \
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
	
    # Added LICENSES files
    mkdir -p "$pkgdir/usr/share/licenses/calamares5/"
    for L in BSD-2-Clause CC-BY-4.0 CC0-1.0 GPL-3.0-or-later LGPL-2.1-only LGPL-3.0-or-later MIT;
    do
	    install -Dm644 "../LICENSES/$L.txt" "$pkgdir/usr/share/licenses/calamares5/$L"
    done
	# fix branding install
	cp -av "../src/branding/manjaro" "$pkgdir/usr/share/calamares/branding/"
}
