# Maintainer: Giovanni Scafora <giovanni@archlinux.org>
# Contributor: Petrov Roman <nwhisper@gmail.com>
# Contributor: Andrea Fagiani <andfagiani _at_ gmail dot com>

pkgname=calibre
pkgver=0.9.15
pkgrel=1
pkgdesc="Ebook management application"
arch=('i686' 'x86_64') 
url="http://calibre-ebook.com/"
license=('GPL3')
depends=('python2-dateutil' 'python2-cssutils' 'python2-cherrypy' 
         'python2-mechanize' 'podofo' 'libwmf' 'python2-beautifulsoup3' 
         'imagemagick' 'poppler-qt' 'chmlib' 'python2-lxml' 'libusbx' 
         'python2-imaging' 'shared-mime-info' 'python2-dnspython' 
         'libunrar' 'python2-pyqt' 'python2-psutil' 'icu' 'libmtp' 
         'python2-netifaces' 'python2-cssselect')
makedepends=('python2-pycountry' 'qt-private-headers')
optdepends=('ipython2: to use calibre-debug')
install=calibre.install
source=("http://calibre-ebook.googlecode.com/files/${pkgname}-${pkgver}.tar.xz"
        'desktop_integration.patch'
        'calibre-mount-helper')
md5sums=('2e87ac93bb2006e40001a4028945cee2'
         '42c07b43d575b5e7e7524bd7b9528f0e'
         '675cd87d41342119827ef706055491e7')

build() {
  cd "${srcdir}/${pkgname}"

  #rm -rf src/{cherrypy,pyPdf}
  rm -rf src/cherrypy
  sed -i -e "s/ldflags = shlex.split(ldflags)/ldflags = shlex.split(ldflags) + ['-fPIC']/" setup/extensions.py

  # Fix for calibre-0.8.58
  sed -i -e "s:#!usr:#!/usr:g" src/calibre/ebooks/markdown/extensions/meta.py

  sed -i -e 's:\(#!/usr/bin/env[ ]\+python$\|#![ ]/usr/bin/env[ ]\+python$\|#!/usr/bin/python$\):\12:g' \
    $(find . -regex ".*.py\|.*.recipe")

  LANG='en_US.UTF-8' python2 setup.py build
  # LANG='en_US.UTF-8' python2 setup.py resources
  LANG='en_US.UTF-8' python2 setup.py translations
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
  LANG='en_US.UTF-8' python2 setup.py install --root="${pkgdir}" --prefix=/usr \
    --staging-bindir="${pkgdir}/usr/bin" \
    --staging-libdir="${pkgdir}/usr/lib" \
    --staging-sharedir="${pkgdir}/usr/share"

  find "${pkgdir}" -type d -empty -delete

  # See http://lwn.net/SubscriberLink/465311/7c299471a5399167/
  rm -rf "${pkgdir}/usr/bin/calibre-mount-helper"
  install -m 755 "${srcdir}/calibre-mount-helper" "${pkgdir}/usr/bin"

  # Compiling bytecode FS33392
  python2 -m compileall "${pkgdir}/usr/lib/calibre/"

  # Compiling optimized bytecode FS33392
  python2 -O -m compileall "${pkgdir}/usr/lib/calibre/"
}
