# Maintainer: Giovanni Scafora <giovanni@archlinux.org>
# Contributor: Petrov Roman <nwhisper@gmail.com>
# Contributor: Andrea Fagiani <andfagiani _at_ gmail dot com>

pkgname=calibre
pkgver=0.7.43
pkgrel=1
pkgdesc="Ebook management application"
arch=('i686' 'x86_64') 
url="http://calibre.kovidgoyal.net/"
license=('GPL3')
depends=('python-dateutil' 'python2-cssutils' 'python-pypdf' 'cherrypy' 
         'python-mechanize' 'podofo' 'libwmf' 'python-beautifulsoup' 
         'imagemagick' 'poppler-qt' 'chmlib' 'python-lxml' 'libusb' 
         'python-imaging' 'desktop-file-utils' 'shared-mime-info' 
         'python-dnspython' 'unrar' 'python2-qt' 'icu')
makedepends=('python2-pycountry')
optdepends=('ipython: to use calibre-debug')
install=calibre.install
source=(http://calibre-ebook.googlecode.com/files/${pkgname}-${pkgver}.tar.gz
        desktop_integration.patch)
md5sums=('28a9792a14b913e629eedd10e88b824d'
         'bcc538a3b004429bf8f5a0ac1d89a37f')

build() {
  cd "${pkgname}"

  rm -rf src/{cherrypy,pyPdf}
  sed -i -e "s/ldflags = shlex.split(ldflags)/ldflags = shlex.split(ldflags) + ['-fPIC']/" setup/extensions.py
  sed -i -e 's:\(#!/usr/bin/env[ ]\+python$\|#!/usr/bin/python$\):\12:g' \
    $(find . -regex ".*.py\|.*.recipe")

  python2 setup.py build || return 1
  python2 setup.py resources || return 1
  python2 setup.py translations || return 1
}

package() {
  cd ${pkgname}
  
  patch -Np1 -i $srcdir/desktop_integration.patch || return 1

  # More on desktop integration (e.g. enforce arch defaults)
  sed -i -e "/self.create_uninstaller()/,/os.rmdir(config_dir)/d" \
      -e "s|self.opts.staging_sharedir, 'man/man1'|self.opts.staging_root, 'usr/share/man/man1'|" \
      -e "s|manpath, prog+'.1'+__appname__+'.bz2'|manpath, prog+'.1'+'.bz2'|" \
      -e "s|old_udev = '/etc|old_udev = '${pkgdir}/etc|" \
      -e "s/^Name=calibre/Name=Calibre/g" src/calibre/linux.py

  # Fix the environment module location
  sed -i -e "s|(prefix=.*)|(prefix='$pkgdir/usr')|g" setup/install.py

  mkdir -p ${pkgdir}/usr/lib/python2.7/site-packages
  python2 setup.py install --root=${pkgdir}/ --prefix=/usr \
    --staging-bindir=${pkgdir}/usr/bin \
    --staging-libdir=${pkgdir}/usr/lib \
    --staging-sharedir=${pkgdir}/usr/share

  find ${pkgdir} -type d -empty -delete

  # Decompress the man pages so makepkg will do it for us.
  for decom in ${pkgdir}/usr/share/man/man1/*.bz2; do
    bzip2 -d ${decom}
  done
}
