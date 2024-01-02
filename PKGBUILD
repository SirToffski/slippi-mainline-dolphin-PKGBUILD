# Maintainer: Daniel Peukert <daniel@peukert.cc>
# Contributor: Maxime Gauduin <alucryd@archlinux.org>
# Contributor: Lightning <sgsdxzy@gmail.com>
# Contributor: Andrew Phelps <andrewphelpsj@gmail.com>
_projectname='slippi-mainline-dolphin'
_mainpkgname="$_projectname-emu"
pkgbase="$_mainpkgname-git"
pkgname=("$pkgbase")
pkgver=v4.0.0.mainline.beta.2.r0.g0c44113f2b
pkgrel=1
pkgdesc='A Gamecube / Wii emulator'
_pkgdescappend=' - git version'
arch=('x86_64' 'aarch64')
url="https://$_mainpkgname.org"
license=('GPL2')
depends=(
	'alsa-lib'
	'bluez-libs'
	'corrosion'
	'cubeb'
	'enet'
	'hicolor-icon-theme'
	'hidapi'
	'libavcodec.so'
	'libavformat.so'
	'libavutil.so'
	'libcurl.so'
	'libevdev'
	'libgl'
	'libminiupnpc.so'
	'libpulse'
	'libsfml-network.so'
	'libsfml-system.so'
	'libspng.so'
	'libswscale.so'
	'libudev.so'
	'libusb-1.0.so'
	'libx11'
	'libxi'
	'libxrandr'
	'libxxhash.so'
	'lzo'
	'mbedtls2'
	'minizip-ng'
	'pugixml'
	'qt5-base'
	'qt5-svg'
	'sfml'
	'speexdsp'
	'xz'
	'zlib-ng'
	'zstd'
)
makedepends=('cmake' 'git' 'ninja' 'python' 'qt5-base' 'qt5-svg' 'miniupnpc' 'cargo')
optdepends=('pulseaudio: PulseAudio backend')
options=('!lto')
source=(
        "$pkgname::git+https://github.com/project-slippi/dolphin.git#commit=0c44113f2b1ce10e730733961529a4eb6a3ffd19"
        "$pkgname-implot::git+https://github.com/epezent/implot.git"
        "$pkgname-mgba::git+https://github.com/mgba-emu/mgba.git"
        "$pkgname-rcheevos::git+https://github.com/RetroAchievements/rcheevos.git"
        "$pkgname-vma::git+https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator.git#commit=c351692490513cdb0e5a2c925aaf7ea4a9b672f4"
        "$pkgname-corrosion::git+https://github.com/corrosion-rs/corrosion.git"
        "$pkgname-slippi-rust-extensions::git+https://github.com/project-slippi/slippi-rust-extensions.git"
        "$pkgname-spirv-cross::git+https://github.com/KhronosGroup/SPIRV-Cross.git"
        '001_patch_to_build.patch'
        '002_fix_more_libs.patch'
        '003_fix_vulkan_gcc13.patch'
)
sha512sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            )
_sourcedirectory="$pkgname"
_dolphinemu="dolphin-emu"

prepare() {
        cd "$srcdir/$_sourcedirectory/"
        if [ -d 'build/' ]; then rm -rf 'build/'; fi
        mkdir 'build/'

        patch --forward -p1 < "$srcdir/001_patch_to_build.patch"
        patch --forward -p1 < "$srcdir/002_fix_more_libs.patch"



        # Provide submodules
        declare -A _submodules=(
                [mgba]='mGBA/mgba'
                [vma]='VulkanMemoryAllocator'
                [corrosion]='corrosion'
                [slippi-rust-extensions]='SlippiRustExtensions'
                [spirv-cross]='spirv_cross'
                [implot]='implot'
        )

        for _submod in "${!_submodules[@]}"; do
                _path="Externals/${_submodules[$_submod]}"
                git submodule init "$_path"
                git config "submodule.$_path.url" "$srcdir/$pkgname-$_submod/"
                git -c protocol.file.allow=always submodule update "$_path"
        done

        patch --forward -p1 < "$srcdir/003_fix_vulkan_gcc13.patch"

}

pkgver() {
        cd "$srcdir/$_sourcedirectory/"
        git describe --long --tags | sed -e 's/-\([^-]*-g[^-]*\)$/-r\1/' -e 's/-/./g'
}

build() {
        CMAKE_FLAGS='-DLINUX_LOCAL_DEV=true -DSLIPPI_PLAYBACK=false -DUSE_SYSTEM_LIBS=ON -DCMAKE_BUILD_TYPE=Release -DUSE_SHARED_ENET=OFF -DENABLE_TESTS=OFF -DENABLE_NOGUI=OFF -DENABLE_CLI_TOOL=OFF -DUSE_MGBA=OFF'
        export LDFLAGS="-Wl,--copy-dt-needed-entries"
        cd "$srcdir/$_sourcedirectory/"
        mkdir -p build
        cd build
        cmake ${CMAKE_FLAGS} ../
        cmake --build . --target dolphin-emu -- -j"$(nproc)"

}

package() {
        pkgdesc="$pkgdesc$_pkgdescappend"
        provides=("$_mainpkgname")
        conflicts=("$_mainpkgname" "slippi-online-git")

        cd "$srcdir/$_sourcedirectory/"
        make DESTDIR="$pkgdir" -C 'build/' install
        #mkdir -p "$pkgdir/usr/local/bin/$_mainpkgname"
        mv "$pkgdir/usr/local/bin/$_dolphinemu" "$pkgdir/usr/local/bin/$_mainpkgname"
        cp -r "Data/Sys/" "$pkgdir/usr/local/bin/"
        rm -r "$pkgdir/usr/local/share/man/"
        #install -Dm644 'Data/51-usb-device.rules' "$pkgdir/usr/lib/udev/rules.d/51-usb-device.rules"
        install -Dm644 'build/cargo/build/x86_64-unknown-linux-gnu/release/libslippi_rust_extensions.so' "$pkgdir/usr/lib/libslippi_rust_extensions.so"
        rm -rf "$pkgdir/usr/include"
        rm -rf "$pkgdir/usr/lib/libdiscord-rpc.a"


}
