# Maintainer: Jelle van der Waa <jelle@vdwaa.nl>
# Maintainer: Daniel Wallace <danielwallace at gtmanfred dot com>
# Contributor: Giovanni Scafora <giovanni@archlinux.org>
# Contributor: Petrov Roman <nwhisper@gmail.com>
# Contributor: Andrea Fagiani <andfagiani _at_ gmail dot com>
# Contributor: Larry Hajali <larryhaja@gmail.com>

pkgname=calibre
pkgver=1.204.0
pkgrel=1
pkgdesc="Ebook management application"
arch=('i686' 'x86_64') 
url="http://calibre-ebook.com/"
license=('GPL3')
depends=('python2-six' 'python2-dateutil' 'python2-cssutils' 'python2-cherrypy' 
         'python2-mechanize' 'podofo' 'libwmf' 'python2-beautifulsoup3' 
         'imagemagick' 'poppler-qt' 'chmlib' 'python2-lxml' 'libusbx' 
         'python2-pillow' 'shared-mime-info' 'python2-dnspython' 
         'libunrar' 'python2-pyqt5' 'python2-psutil' 'pyqt4-common' 'icu' 'libmtp' 
         'python2-netifaces' 'python2-cssselect' 'python2-apsw' 'qt5-webkit')
makedepends=('python2-pycountry' 'qt5-x11extras' )
optdepends=('ipython2: to use calibre-debug')
install=calibre.install
source=("http://download.calibre-ebook.com/betas/calibre-${pkgver}.tar.xz"
        'desktop_integration.patch'
        'fix_sip.patch')

prepare(){
  cd "${srcdir}/${pkgname}-${pkgver}"
  #rm -rf src/{cherrypy,pyPdf}
  rm -rf src/cherrypy
  rm -rf resources/${pkgname}-portable.*
  sed -i -e "s/ldflags = shlex.split(ldflags)/ldflags = shlex.split(ldflags) + ['-fPIC']/" setup/extensions.py

  # Fix for calibre-0.8.58
  sed -i -e "s:#!usr:#!/usr:g" src/calibre/ebooks/markdown/extensions/meta.py

  sed -i -e 's:\(#!/usr/bin/env[ ]\+python$\|#![ ]/usr/bin/env[ ]\+python$\|#!/usr/bin/python$\):\12:g' \
    $(find . -regex ".*.py\|.*.recipe")

  patch -Np0 -i $srcdir/fix_sip.patch
}

build() {
  cd "${srcdir}/${pkgname}-${pkgver}"

  LANG='en_US.UTF-8' python2 setup.py build
  # LANG='en_US.UTF-8' python2 setup.py resources

  # Don't build translations since building them is broken badly
  #LANG='en_US.UTF-8' python2 setup.py translations
}

package() {
  cd "${srcdir}/${pkgname}-${pkgver}"
  
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
  install -d "${pkgdir}/usr/share/zsh/site-functions"
  LANG='en_US.UTF-8' python2 setup.py install --root="${pkgdir}" --prefix=/usr \
    --staging-bindir="${pkgdir}/usr/bin" \
    --staging-libdir="${pkgdir}/usr/lib" \
    --staging-sharedir="${pkgdir}/usr/share"

  find "${pkgdir}" -type d -empty -delete

  # Compiling bytecode FS33392
  python2 -m compileall "${pkgdir}/usr/lib/calibre/"

  # Compiling optimized bytecode FS33392
  python2 -O -m compileall "${pkgdir}/usr/lib/calibre/"
}

md5sums=('379492d52b389debbad17af137f6e5e6'
         '52c8bc5103ec6f06c485eac6b79124b3'
         '675cd87d41342119827ef706055491e7'
         'b4f759b533977eb6b728892310d2e48a')
md5sums=('379492d52b389debbad17af137f6e5e6'
         '52c8bc5103ec6f06c485eac6b79124b3'
         'b4f759b533977eb6b728892310d2e48a')
