# Maintainer: soh @ AUR
# Maintainer: Tobias Kunze <r@rixx.de>
# Maintained at https://github.com/rixx/pkgbuilds, feel free to submit patches

pkgname=python311
pkgver=3.11.11
pkgrel=0
_pybasever=3.11
_pymajver=3
pkgdesc="Major release 3.11 of the Python high-level programming language"
arch=('i686' 'x86_64')
license=('PSF-2.0')
url="https://www.python.org/"
depends=('bzip2' 'expat' 'gdbm' 'libffi' 'libnsl' 'libxcrypt' 'openssl' 'zlib' 'tzdata' 'mpdecimal')
makedepends=('tk' 'sqlite' 'bluez-libs' 'llvm' 'gdb' 'xorg-server-xvfb' 'ttf-font')
source=(https://www.python.org/ftp/python/${_pyver}/Python-${pkgver}.tar.xz)
sha256sums=('2a9920c7a0cd236de33644ed980a13cbbc21058bfdc528febb6081575ed73be3')
validpgpkeys=(
    '0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D'  # Ned Deily (Python release signing key) <nad@python.org>
    'E3FF2839C048B25C084DEBE9B26995E310250568'  # Łukasz Langa (GPG langa.pl) <lukasz@langa.pl>
)
provides=("python=$pkgver")

prepare() {
  cd "${srcdir}/Python-${pkgver}"

  # FS#23997
  sed -i -e "s|^#.* /usr/local/bin/python|#!/usr/bin/python|" Lib/cgi.py

  # Ensure that we are using the system copy of various libraries (expat, zlib and libffi),
  # rather than copies shipped in the tarball
  rm -rf Modules/expat
  rm -rf Modules/zlib
  rm -rf Modules/_ctypes/{darwin,libffi}*
  rm -rf Modules/_decimal/libmpdec
}

build() {
  cd "${srcdir}/Python-${pkgver}"

  CFLAGS="${CFLAGS} -fno-semantic-interposition -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer"
  CFLAGS="${CFLAGS/-O2/-O3} -ffat-lto-objects"
  CFLAGS="${CFLAGS} -march=znver4 -mtune=znver4"
  ./configure ax_cv_c_float_words_bigendian=no \
              --prefix=/usr \
              --enable-shared \
              --with-computed-gotos \
              --enable-optimizations \
              --with-lto \
              --enable-ipv6 \
              --with-system-expat \
              --with-dbmliborder=gdbm:ndbm \
              --with-system-libmpdec \
              --enable-loadable-sqlite-extensions \
              --without-ensurepip \
              --with-tzpath=/usr/share/zoneinfo

  # Obtain next free server number for xvfb-run; this even works in a chroot environment.
  export servernum=99
  while ! xvfb-run -a -n "$servernum" /bin/true 2>/dev/null; do servernum=$((servernum+1)); done

  # Build
  LC_CTYPE=en_US.UTF-8 xvfb-run -s "-screen 0 1920x1080x16 -ac +extension GLX" -a -n "$servernum" make EXTRA_CFLAGS="$CFLAGS"
}

package() {
  cd "${srcdir}/Python-${pkgver}"
  # altinstall: /usr/bin/pythonX.Y but not /usr/bin/python or /usr/bin/pythonX
  make DESTDIR="${pkgdir}" altinstall maninstall

  # Split tests
  rm -r "$pkgdir"/usr/lib/python*/{test,idlelib/idle_test}

  # Avoid conflicts with the main 'python' package.
  rm -f "${pkgdir}/usr/lib/libpython${_pymajver}.so"
  rm -f "${pkgdir}/usr/share/man/man1/python${_pymajver}.1"

  # Clean-up reference to build directory
  sed -i "s|$srcdir/Python-${pkgver}:||" "$pkgdir/usr/lib/python${_pybasever}/config-${_pybasever}-${CARCH}-linux-gnu/Makefile"

  # Add useful scripts FS#46146
  install -dm755 "${pkgdir}"/usr/lib/python${_pybasever}/Tools/{i18n,scripts}
  install -m755 Tools/i18n/{msgfmt,pygettext}.py "${pkgdir}"/usr/lib/python${_pybasever}/Tools/i18n/
  install -m755 Tools/scripts/{README,*py} "${pkgdir}"/usr/lib/python${_pybasever}/Tools/scripts/

  # License
  install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
