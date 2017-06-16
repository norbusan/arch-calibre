# Maintainer: Jelle van der Waa <jelle@vdwaa.nl>
# Maintainer: Daniel Wallace <danielwallace at gtmanfred dot com>
# Contributor: Giovanni Scafora <giovanni@archlinux.org>
# Contributor: Petrov Roman <nwhisper@gmail.com>
# Contributor: Andrea Fagiani <andfagiani _at_ gmail dot com>
# Contributor: Larry Hajali <larryhaja@gmail.com>
# Contributor: Eli Schwartz <eschwartz93@gmail.com>

pkgname=calibre
pkgver=3.0.0
pkgrel=1
pkgdesc="Ebook management application"
arch=('i686' 'x86_64')
url="https://calibre-ebook.com/"
license=('GPL3')
depends=('python2-six' 'python2-dateutil' 'python2-cssutils' 'python2-dukpy'
         'python2-mechanize' 'podofo' 'libwmf'
         'chmlib' 'python2-lxml' 'libusbx'
         'python2-pillow' 'shared-mime-info' 'python2-dnspython'
         'python2-pyqt5' 'python2-psutil' 'icu' 'libmtp' 'python2-dbus'
         'python2-netifaces' 'python2-cssselect' 'python2-apsw' 'qt5-webkit'
         'qt5-svg' 'python2-regex' 'python2-pygments' 'mtdev'
         'desktop-file-utils' 'gtk-update-icon-cache' 'optipng' 'udisks2')
makedepends=('qt5-x11extras' 'xdg-utils')
optdepends=('ipython2: to use calibre-debug'
            'poppler: required for converting pdf to html'
)
source=("https://download.calibre-ebook.com/${pkgver}/calibre-${pkgver}.tar.xz"
        "https://calibre-ebook.com/signatures/${pkgname}-${pkgver}.tar.xz.sig")
sha256sums=('7cdaa5c55c8c13d6b5389ee11231c64944f5272297938e426be3a4fcf63a13fe'
            'SKIP')
validpgpkeys=('3CE1780F78DD88DF45194FD706BC317B515ACE7C')

prepare(){
  cd "${pkgname}-${pkgver}"

  # Remove unneeded files and libs
  # FIXME: remove html5lib too, but the latest html5lib breaks calibre. And calibre
  # really needs to get the latest changes into html5lib, cause forking is bad m'okay.
  rm -rf resources/${pkgname}-portable.*

  # Desktop integration (e.g. enforce arch defaults)
  sed -e "/self.create_uninstaller()/,/os.rmdir(config_dir)/d" \
      -e "/cc(\['xdg-desktop-menu', 'forceupdate'\])/d" \
      -e "/cc(\['xdg-mime', 'install', MIME\])/d" \
      -e "s/'ctc-posml'/'text' not in mt and 'pdf' not in mt and 'xhtml'/" \
      -e "s/^Name=calibre/Name=Calibre/g" \
      -i  src/calibre/linux.py
}

build() {
  cd "${pkgname}-${pkgver}"

  LANG='en_US.UTF-8' python2 setup.py build
  LANG='en_US.UTF-8' python2 setup.py gui
}

package() {
  cd "${pkgname}-${pkgver}"

  install -d "${pkgdir}/usr/share/zsh/site-functions" \
             "${pkgdir}"/usr/share/{applications,desktop-directories,icons/hicolor}

  install -Dm644 resources/calibre-mimetypes.xml \
    "${pkgdir}/usr/share/mime/packages/calibre-mimetypes.xml"

  XDG_DATA_DIRS="${pkgdir}/usr/share" LANG='en_US.UTF-8' \
    python2 setup.py install --staging-root="${pkgdir}/usr" --prefix=/usr

  # Compiling bytecode FS#33392
  python2 -m compileall "${pkgdir}/usr/lib/calibre/"
  python2 -O -m compileall "${pkgdir}/usr/lib/calibre/"
}
