# Maintainer: artist for Artix Linux

pkgbase=xlibre-xserver
pkgname=($pkgbase $pkgbase-xephyr $pkgbase-xvfb $pkgbase-xnest $pkgbase-common $pkgbase-devel)
pkgver=25.1.0
pkgrel=6
arch=('x86_64')
license=('LicenseRef-Adobe-Display-PostScript'
         'BSD-3-Clause' 
         'LicenseRef-DEC-3-Clause' 
         'HPND'
         'LicenseRef-HPND-sell-MIT-disclaimer-xserver'
         'HPND-sell-variant' 
         'ICU'
         'ISC'
         'MIT'
         'MIT-open-group'
         'NTP'
         'SGI-B-2.0'
         'SMLNJ'
         'X11'
         'X11-distribute-modifications-variant')
groups=('xlibre')
url="https://github.com/X11Libre/xserver"
makedepends=('xorgproto' 'pixman' 'libx11' 'mesa' 'mesa-libgl'
             'libxkbfile' 'libxfont2' 'libpciaccess' 'libxv' 'libxcvt'
             'libxmu' 'libxrender' 'libxi' 'libxaw' 'libxtst' 'libxres'
             'xorg-xkbcomp' 'xorg-util-macros' 'xorg-font-util' 'libepoxy'
             'xcb-util' 'xcb-util-image' 'xcb-util-renderutil' 'xcb-util-wm' 'xcb-util-keysyms'
             'libxshmfence' 'libunwind' 'meson' 'elogind')
source=("${url}/archive/refs/tags/${pkgname}-${pkgver}.tar.gz"
        xvfb-run # with updates from FC master
        xvfb-run.1)

build() {
  export CFLAGS=${CFLAGS/-fno-plt}
  export CXXFLAGS=${CXXFLAGS/-fno-plt}
  export LDFLAGS=${LDFLAGS/-Wl,-z,now}

  artix-meson xserver-${pkgbase}-${pkgver} build \
    --buildtype=release \
    -D ipv6=true \
    -D xvfb=true \
    -D xnest=true \
    -D xcsecurity=true \
    -D xorg=true \
    -D xephyr=true \
    -D glamor=true \
    -D udev=true \
    -D udev_kms=true \
    -D dtrace=false \
    -D systemd_logind=true \
    -D suid_wrapper=true \
    -D linux_acpi=false \
    -D legacy_nvidia_padding=true \
    -D legacy_nvidia_340x=true \
    -D xkb_dir=/usr/share/X11/xkb \
    -D xkb_output_dir=/var/lib/xkb \
    -D libunwind=true 

  meson configure build
  ninja -C build

  DESTDIR="${srcdir}/fakeinstall" ninja -C build install
}

_install() {
  local src f dir
  for src; do
    f="${src#fakeinstall/}"
    dir="${pkgdir}/${f%/*}"
    install -m755 -d "${dir}"
    # use copy so a new file is created and fakeroot can track properties such as setuid
    cp -av "${src}" "${dir}/"
    rm -rf "${src}"
  done
}

package_xlibre-xserver-common() {
  pkgdesc="XLibre fork of X.Org Xorg server common files"
  depends=(xkeyboard-config xorg-xkbcomp xorg-setxkbmap)
  provides=('xorg-server-common')
  conflicts=('xorg-server-common')

  _install fakeinstall/usr/lib/xorg/protocol.txt
  _install fakeinstall/usr/share/man/man1/Xserver.1

  install -m644 -Dt "${pkgdir}/var/lib/xkb/" "xserver-${pkgbase}-${pkgver}"/xkb/README.compiled
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "xserver-${pkgbase}-${pkgver}"/COPYING
}

package_xlibre-xserver() {
  pkgdesc="XLibre fork of X.Org X server"
  depends=("xlibre-xserver-common=${pkgver}" xlibre-input-libinput libepoxy libxfont2 pixman libunwind
           dbus libgl nettle libxdmcp sh glibc libxau libtirpc libbsd
           libpciaccess libdrm libxshmfence libxcvt) # FS#52949
  provides=('xorg-server' 'X-ABI-VIDEODRV_VERSION=28.0' 'X-ABI-XINPUT_VERSION=26.0' 'X-ABI-EXTENSION_VERSION=11.0' 'x-server' 'x11win-server')
  conflicts=('xorg-server' 'nvidia-utils<=331.20' 'glamor-egl' 'xf86-video-modesetting')
  replaces=('glamor-egl' 'xf86-video-modesetting')
  install=xlibre-xserver.install

  _install fakeinstall/usr/bin/{X,Xorg,gtf}
  _install fakeinstall/usr/lib/Xorg{,.wrap}
  _install fakeinstall/usr/lib/xorg/modules/*
  _install fakeinstall/usr/share/X11/xorg.conf.d/10-nvidia.conf
  _install fakeinstall/usr/share/X11/xorg.conf.d/10-quirks.conf
  _install fakeinstall/usr/share/man/man1/{Xorg,Xorg.wrap,gtf}.1
  _install fakeinstall/usr/share/man/man4/{exa,fbdevhw,inputtestdrv,modesetting}.4
  _install fakeinstall/usr/share/man/man5/{Xwrapper.config,xorg.conf,xorg.conf.d}.5

  install -m755 -d "${pkgdir}/etc/X11/xorg.conf.d"

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "xserver-${pkgbase}-${pkgver}"/COPYING
}

package_xlibre-xserver-xephyr() {
  pkgdesc="XLibre fork of X.Org nested X server that runs as an X application"
  depends=(xlibre-xserver-common=${pkgver} libxfont2 libgl libepoxy libunwind
           xcb-util-image xcb-util-renderutil xcb-util-wm xcb-util-keysyms pixman
           nettle libtirpc xcb-util libxdmcp libx11 libxau libxshmfence glibc)
  provides=('xorg-server-xephyr')
  conflicts=('xorg-server-xephyr')

  _install fakeinstall/usr/bin/Xephyr
  _install fakeinstall/usr/share/man/man1/Xephyr.1

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "xserver-${pkgbase}-${pkgver}"/COPYING
}

package_xlibre-xserver-xvfb() {
  pkgdesc="XLibre fork of X.Org virtual framebuffer X server"
  # xvfb-run is GPLv2, rest is MIT
  license=('MIT' 'GPL-2.0-only')
  depends=("xlibre-xserver-common=${pkgver}" libxfont2 libunwind pixman xorg-xauth 
           libgl nettle libtirpc
           libxdmcp sh glibc libxau)
  provides=('xorg-server-xvfb')
  conflicts=('xorg-server-xvfb')

  _install fakeinstall/usr/bin/Xvfb
  _install fakeinstall/usr/share/man/man1/Xvfb.1

  install -m755 "${srcdir}/xvfb-run" "${pkgdir}/usr/bin/"
  install -m644 "${srcdir}/xvfb-run.1" "${pkgdir}/usr/share/man/man1/" # outda

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "xserver-${pkgbase}-${pkgver}"/COPYING
}

package_xlibre-xserver-xnest() {
  pkgdesc="XLibre fork of X.Org nested X server that runs as an X application"
  depends=("xlibre-xserver-common=${pkgver}" libxfont2 libunwind libxext pixman nettle
           libtirpc
           libxdmcp glibc libx11 libxau)
  provides=('xorg-server-xnest')
  conflicts=('xorg-server-xnest')

  _install fakeinstall/usr/bin/Xnest
  _install fakeinstall/usr/share/man/man1/Xnest.1

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "xserver-${pkgbase}-${pkgver}"/COPYING
}

package_xlibre-xserver-devel() {
  pkgdesc="XLibre fork of X.Org development files for the X.Org X server"
  depends=("xlibre-xserver=${pkgver}" 'xorgproto' 'mesa' 'libpciaccess' 'pixman'
           # not technically required but almost every Xorg pkg needs it to build
           'xorg-util-macros')
  provides=('xorg-server-devel')
  conflicts=('xorg-server-devel')

  _install fakeinstall/usr/include/xorg/*
  _install fakeinstall/usr/lib/pkgconfig/xorg-server.pc
  _install fakeinstall/usr/share/aclocal/xorg-server.m4
  _install fakeinstall/usr/lib/pkgconfig/xlibre-server.pc

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "xserver-${pkgbase}-${pkgver}"/COPYING

  find fakeinstall -depth -print0 | xargs -0 rmdir
}
sha256sums=('b46fe85e306821504a5e0e91aa0d8d54ce7ddfb67106c495a6ca5ac2c3e28f06'
            '27ce50f4432e5549e662db857118761fa9cd74c6900aac52c4db768c956838db'
            '2460adccd3362fefd4cdc5f1c70f332d7b578091fb9167bf88b5f91265bbd776')
