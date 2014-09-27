# Maintainer: Jelle van der Waa <jelle@vdwaa.nl>
# Maintainer: Daniel Wallace <danielwallace at gtmanfred dot com>
# Contributor: Giovanni Scafora <giovanni@archlinux.org>
# Contributor: Petrov Roman <nwhisper@gmail.com>
# Contributor: Andrea Fagiani <andfagiani _at_ gmail dot com>
# Contributor: Larry Hajali <larryhaja@gmail.com>

pkgname=calibre
pkgver=2.4.0
pkgrel=1
pkgdesc="Ebook management application"
arch=('i686' 'x86_64') 
url="http://calibre-ebook.com/"
license=('GPL3')
depends=('python2-six' 'python2-dateutil' 'python2-cssutils' 'python2-cherrypy' 
         'python2-mechanize' 'podofo' 'libwmf'
         'imagemagick' 'chmlib' 'python2-lxml' 'libusbx' 
         'python2-pillow' 'shared-mime-info' 'python2-dnspython' 
         'python2-pyqt5' 'python2-psutil' 'icu' 'libmtp' 
         'python2-netifaces' 'python2-cssselect' 'python2-apsw' 'qt5-webkit'
         'qt5-svg' 'python2-chardet' 'python2-html5lib')
makedepends=('python2-pycountry' 'qt5-x11extras' )
optdepends=('ipython2: to use calibre-debug'
            'udisks: required for mounting certain devices'
)
install=calibre.install
source=("http://download.calibre-ebook.com/${pkgver}/calibre-${pkgver}.tar.xz")
md5sums=('fc8f9b03879b959aa7d18b460f285eeb')

prepare(){
  cd "${srcdir}/${pkgname}-${pkgver}"

  # Remove unneeded files and libs
  rm -rf resources/${pkgname}-portable.* \
         src/six.py \
         src/cherrypy \
         src/html5lib \
         src/chardet

  sed -i "s/shlex.split(ldflags)/& + ['-fPIC']/" setup/extensions.py
  #sed -i -e "s/ldflags = shlex.split(ldflags)/ldflags = shlex.split(ldflags) + ['-fPIC']/" setup/extensions.py

  # Use python2
  sed -i 's:\(env[ ]\+python$\|/usr/bin/python$\):\12:g' $(find . -regex ".*\.py\|.*\.recipe")
  sed -i "/pyqt_sip_dir/ s:=.*:= '/usr/share/sip/Py2-PyQt5':" setup/build_environment.py

  # Desktop integration (e.g. enforce arch defaults)
  sed -e "/self.create_uninstaller()/,/os.rmdir(config_dir)/d" \
      -e "/\(cc('xdg-icon-resource\|self.icon_resources.append\|'128'))\)/d" \
      -e "/render_img/ s/\('calibre-.*\.png'\)/os.path.join(dir, \1)/g" \
      -e "/dir, 'calibre-lrf.png'/i \
\                dir = os.path.join(self.opts.staging_sharedir,'../pixmaps')\n\
\                os.mkdir(dir)" \
      -e "/f = open/ s/\('calibre-.*\.desktop'\)/os.path.join(dir, \1)/g" \
      -e "/dir, 'calibre-lrfviewer.desktop'/i \
\                dir = os.path.join(self.opts.staging_sharedir,'../applications')\n\
\                os.mkdir(dir)" \
      -e "s/'ctc-posml'/'text' not in mt and 'pdf' not in mt and 'xhtml'/" \
      -e "s/^Name=calibre/Name=Calibre/g" \
      -i  src/calibre/linux.py
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
  
  # Fix the environment module location
  sed -i -e "s|(prefix=.*)|(prefix='$pkgdir/usr')|g" setup/install.py

  install -d "${pkgdir}/usr/lib/python2.7/site-packages" \
             "${pkgdir}/usr/share/zsh/site-functions"

  LANG='en_US.UTF-8' python2 setup.py install --root="${pkgdir}" --prefix=/usr \
    --staging-bindir="${pkgdir}/usr/bin" \
    --staging-libdir="${pkgdir}/usr/lib" \
    --staging-sharedir="${pkgdir}/usr/share"

  # Compiling bytecode FS#33392
  python2 -m compileall "${pkgdir}/usr/lib/calibre/"
  python2 -O -m compileall "${pkgdir}/usr/lib/calibre/"
}
