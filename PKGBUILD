# Maintainer: Caleb Maclennan <caleb@alerque.com>
# Contributor: Jelle van der Waa <jelle@vdwaa.nl>
# Contributor: Eli Schwartz <eschwartz@archlinux.org>
# Contributor: Daniel Wallace <danielwallace at gtmanfred dot com>
# Contributor: Giovanni Scafora <giovanni@archlinux.org>
# Contributor: Petrov Roman <nwhisper@gmail.com>
# Contributor: Andrea Fagiani <andfagiani _at_ gmail dot com>
# Contributor: Larry Hajali <larryhaja@gmail.com>

pkgname=calibre
pkgver=5.38.0
pkgrel=1
pkgdesc='Ebook management application'
arch=(x86_64)
url=https://calibre-ebook.com
license=(GPL3)
_pydeps=(apsw
          beautifulsoup4
          cchardet
          css-parser
          cssselect
          dateutil
          dnspython
          feedparser
          html2text
          html5-parser
          jeepney
          lxml
          markdown
          mechanize
          msgpack
          netifaces
          pillow
          psutil
          py7zr
          pychm
          pygments
          pyqt5
          pyqtwebengine
          regex
          unrardll
          zeroconf)
depends=(hunspell
         hyphen
         icu
         jxrlib
         libmtp
         libstemmer
         libusb
         libwmf
         mathjax
         mtdev
         optipng
         podofo
         "${_pydeps[@]/#/python-}"
         qt5-imageformats
         qt5-svg
         ttf-liberation
         udisks2)
makedepends=(pyqt-builder
             qt5-x11extras
             rapydscript-ng
             sip
             xdg-utils)
optdepends=('poppler: required for converting pdf to html'
            'speech-dispatcher: TTS support in the viewer')
conflicts=(calibre-common
           calibre-python3)
replaces=("${replaces[@]}")
_archive="$pkgname-$pkgver"
source=("https://download.calibre-ebook.com/$pkgver/$_archive.tar.xz"
        "$url/signatures/$_archive.tar.xz.sig")
sha256sums=('82b46b2c9d0ba2c82d95940e285b1fa168d55998613d216fa6d60da7d68fc550'
            'SKIP')
validpgpkeys=('3CE1780F78DD88DF45194FD706BC317B515ACE7C') # Kovid Goyal (New longer key) <kovid@kovidgoyal.net>

prepare(){
	cd "$_archive"

	# Desktop integration (e.g. enforce arch defaults)
	# Use uppercase naming scheme, don't delete config files under fakeroot.
	sed -e "/import config_dir/,/os.rmdir(config_dir)/d" \
		-e "s/'ctc-posml'/'text' not in mt and 'pdf' not in mt and 'xhtml'/" \
		-e "s/^Name=calibre/Name=Calibre/g" \
		-i  src/calibre/linux.py

	cd resources

	# Remove unneeded files
	rm $pkgname-portable.* mozilla-ca-certs.pem

	# use system mathjax
	rm -r mathjax
}

build() {
	cd "$_archive"

	LANG='en_US.UTF-8' python setup.py build
	LANG='en_US.UTF-8' python setup.py gui
	python setup.py liberation_fonts --path-to-liberation_fonts /usr/share/fonts/liberation --system-liberation_fonts
	LANG='en_US.UTF-8' python setup.py mathjax --path-to-mathjax /usr/share/mathjax --system-mathjax
	LANG='en_US.UTF-8' python setup.py rapydscript
}

check() {
	cd "$_archive"

	_test_excludes=(
		# merely testing if a runtime-optional feature optdepend is importable
		'speech_dispatcher'
		# tests if a completely unused module is bundled
		'pycryptodome'
		# only fails on local builds, and that intermittently
		'test_searching'
	)

	LANG='en_US.UTF-8' python setup.py test "${_test_excludes[@]/#/--exclude-test-name=}"
}

package() {
	cd "$_archive"

	# If this directory doesn't exist, zsh completion won't install.
	install -d "${pkgdir}/usr/share/zsh/site-functions"

	LANG='en_US.UTF-8' python setup.py install \
		--staging-root="${pkgdir}/usr" \
		--prefix=/usr \
		--system-plugins-location=/usr/share/calibre/system-plugins

	cp -a man-pages/ "${pkgdir}/usr/share/man"

	# not needed at runtime
	rm -r "${pkgdir}"/usr/share/calibre/rapydscript/

	# Compiling bytecode FS#33392
	# This is kind of ugly but removes traces of the build root.
	while read -rd '' _file; do
		_destdir="$(dirname "${_file#${pkgdir}}")"
		python -m compileall -d "${_destdir}" "${_file}"
		python -O -m compileall -d "${_destdir}" "${_file}"
	done < <(find "${pkgdir}"/usr/lib/ -name '*.py' -print0)
}
