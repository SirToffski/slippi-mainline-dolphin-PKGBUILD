_projectname='slippi-mainline-dolphin'
_mainpkgname="$_projectname-emu"
pkgbase="$_mainpkgname-git"
pkgname=("$pkgbase")
pkgver=v4.0.0.mainline.beta.2.r1.g18c25b2385
pkgrel=1
pkgdesc='A Gamecube / Wii emulator'
_pkgdescappend=' - git version'
arch=('x86_64' 'aarch64')
url="https://$_mainpkgname.org"
license=('GPL2')
depends=(
        'alsa-lib' 'bluez-libs' 'cubeb' 'enet' 'hidapi' 'libevdev' 'libgl' 'libpulse'
        'libspng' 'libx11' 'libxi' 'libxrandr' 'lzo' 'mbedtls2' 'minizip-ng' 'pugixml'
        'qt6-base' 'qt6-svg' 'qt5-base' 'sfml' 'zlib-ng'
        'libavcodec.so' 'libavformat.so' 'libavutil.so' 'libcurl.so' 'libminiupnpc.so'
        'libsfml-network.so' 'libsfml-system.so' 'libswscale.so' 'libudev.so'
        'libusb-1.0.so' 'corrosion'
)
makedepends=('cmake' 'git' 'ninja' 'python')
optdepends=('pulseaudio: PulseAudio backend')
#options=('!lto')
source=(
        "$pkgname::git+https://github.com/project-slippi/dolphin.git"
        "$pkgname-implot::git+https://github.com/epezent/implot.git"
        "$pkgname-mgba::git+https://github.com/mgba-emu/mgba.git"
        "$pkgname-rcheevos::git+https://github.com/RetroAchievements/rcheevos.git"
        "$pkgname-vma::git+https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator.git#commit=c351692490513cdb0e5a2c925aaf7ea4a9b672f4"
        "$pkgname-corrosion::git+https://github.com/corrosion-rs/corrosion.git"
        "$pkgname-slippi-rust-extensions::git+https://github.com/project-slippi/slippi-rust-extensions.git"
        "$pkgname-spirv-cross::git+https://github.com/KhronosGroup/SPIRV-Cross.git"
        'patch_to_build.patch'
        'fix_libs.patch'
        'fix_more_libs.patch'
        'fix_vulkan_gcc13.patch'
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
            'SKIP'
            )

_sourcedirectory="$pkgname"
_dolphinemu="dolphin-emu"

prepare() {
        cd "$srcdir/$_sourcedirectory/"
        if [ -d 'build/' ]; then rm -rf 'build/'; fi
        mkdir 'build/'
        
        patch --forward -p1 < "$srcdir/patch_to_build.patch"
        patch --forward -p1 < "$srcdir/fix_more_libs.patch"
        
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

        patch --forward -p1 < "$srcdir/fix_vulkan_gcc13.patch"
}

pkgver() {
        cd "$srcdir/$_sourcedirectory/"
        git describe --long --tags | sed -e 's/-\([^-]*-g[^-]*\)$/-r\1/' -e 's/-/./g'
}

build() {
        # CMAKE_BUILD_TYPE - the dolphin-emu package in the repos uses 'None' for some reason, so we use it as well
        # USE_SYSTEM_LIBS - we want to use system libs where possible
        # USE_SYSTEM_LIBMGBA - the current version of mgba in the repos is not compatible with Dolphin
        # DUSE_SYSTEM_FMT - the current version of fmt in the repos is not compatible with Dolphin
        CMAKE_FLAGS='-DLINUX_LOCAL_DEV=true -DSLIPPI_PLAYBACK=false -DUSE_SYSTEM_LIBS=ON -DCMAKE_BUILD_TYPE=None -DUSE_SHARED_ENET=ON -DENABLE_TESTS=OFF -DENABLE_NOGUI=OFF -DENABLE_CLI_TOOL=OFF -DUSE_MGBA=ON -DUSE_FMT=ON'
        cd "$srcdir/$_sourcedirectory/"
        mkdir -p build
        cd build
        cmake ${CMAKE_FLAGS} ../
        cmake --build . --target dolphin-emu -- -j$(nproc)
        #cmake -S . -B build -G Ninja \
        #        # -DCMAKE_INSTALL_PREFIX='/usr' \
        #        # -DDISTRIBUTOR='aur.archlinux.org/packages/dolphin-emu-git' \
        #        -DENABLE_TESTS=OFF \
        #        -DUSE_SYSTEM_LIBS=ON \
        #        #-DUSE_MGBA=ON \
        #        #-DUSE_FMT=ON \
        #        -DUSE_SHARED_ENET=ON \
        #        -DSLIPPI_PLAYBACK=false \
        #        -Wno-dev
        #cmake --build 'build'
}

package() {
        pkgdesc="$pkgdesc$_pkgdescappend"
        provides=("$_mainpkgname")
        conflicts=("$_mainpkgname")

        cd "$srcdir/$_sourcedirectory/"
        #DESTDIR="$pkgdir" cmake --install 'build/'
        make DESTDIR="$pkgdir" -C 'build/' install
        mkdir -p "$pkgdir/usr/local/bin/$_mainpkgname"
        mv "$pkgdir/usr/local/bin/$_dolphinemu" "$pkgdir/usr/local/bin/$_mainpkgname/$_mainpkgname"
        cp -r "Data/Sys/" "$pkgdir/usr/local/bin/$_mainpkgname/"
        rm -r "$pkgdir/usr/local/share/man/"
        install -Dm644 'Data/51-usb-device.rules' "$pkgdir/usr/lib/udev/rules.d/51-usb-device.rules"
        install -Dm644 'build/cargo/build/x86_64-unknown-linux-gnu/release/libslippi_rust_extensions.so' "$pkgdir/usr/lib/libslippi_rust_extensions.so"
        rm -rf "$pkgdir/usr/include"
        rm -rf "$pkgdir/usr/lib/libdiscord-rpc.a"
        

}
