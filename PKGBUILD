# Maintainer: Giovanni Scafora <giovanni@archlinux.org>
# Contributor: Petrov Roman <nwhisper@gmail.com>
# Contributor: Andrea Fagiani <andfagiani _at_ gmail dot com>

pkgname=calibre
pkgver=0.8.20
pkgrel=1
pkgdesc="Ebook management application"
arch=('i686' 'x86_64') 
url="http://calibre-ebook.com/"
license=('GPL3')
depends=('python2-dateutil' 'python2-cssutils' 'python2-cherrypy' 
         'python-mechanize' 'podofo' 'libwmf' 'python-beautifulsoup' 
         'imagemagick' 'poppler-qt' 'chmlib' 'python-lxml' 'libusb' 
         'python-imaging' 'desktop-file-utils' 'shared-mime-info' 
         'python-dnspython' 'unrar' 'python2-pyqt' 'icu')
makedepends=('python2-pycountry')
optdepends=('ipython: to use calibre-debug')
install=calibre.install
source=("http://downloads.sourceforge.net/${pkgname}/${pkgname}-${pkgver}.tar.gz"
        'desktop_integration.patch')
md5sums=('1b14bd08233302e3475d9dca0e0941e7'
         '253ce4fe5d01f8ff76b63cd3825755ea')

build() {
  cd "${srcdir}/${pkgname}"

  #rm -rf src/{cherrypy,pyPdf}
  rm -rf src/cherrypy
  sed -i -e "s/ldflags = shlex.split(ldflags)/ldflags = shlex.split(ldflags) + ['-fPIC']/" setup/extensions.py
  sed -i -e 's:\(#!/usr/bin/env[ ]\+python$\|#!/usr/bin/python$\):\12:g' \
    $(find . -regex ".*.py\|.*.recipe")

  python2 setup.py build
  python2 setup.py resources
  python2 setup.py translations
}

package() {
  cd "${srcdir}/${pkgname}"
  
  patch -Np1 -i "${srcdir}/desktop_integration.patch"

  # More on desktop integration (e.g. enforce arch defaults)
  sed -i -e "/self.create_uninstaller()/,/os.rmdir(config_dir)/d" \
      -e "s|self.opts.staging_sharedir, 'man/man1'|self.opts.staging_root, 'usr/share/man/man1'|" \
      -e "s|manpath, prog+'.1'+__appname__+'.bz2'|manpath, prog+'.1'+'.bz2'|" \
      -e "s|old_udev = '/etc|old_udev = '${pkgdir}/etc|" \
      -e "s/^Name=calibre/Name=Calibre/g" src/calibre/linux.py

  # Fix the environment module location
  sed -i -e "s|(prefix=.*)|(prefix='$pkgdir/usr')|g" setup/install.py

  install -d "${pkgdir}/usr/lib/python2.7/site-packages"
  python2 setup.py install --root="${pkgdir}" --prefix=/usr \
    --staging-bindir="${pkgdir}/usr/bin" \
    --staging-libdir="${pkgdir}/usr/lib" \
    --staging-sharedir="${pkgdir}/usr/share"

  find "${pkgdir}" -type d -empty -delete

  # Decompress the man pages so makepkg will do it for us.
  for decom in "${pkgdir}"/usr/share/man/man1/*.bz2; do
    bzip2 -d "${decom}"
  done
}
