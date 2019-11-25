# Maintainer: Jelle van der Waa <jelle@vdwaa.nl>
# Maintainer: Eli Schwartz <eschwartz@archlinux.org>
# Contributor: Daniel Wallace <danielwallace at gtmanfred dot com>
# Contributor: Giovanni Scafora <giovanni@archlinux.org>
# Contributor: Petrov Roman <nwhisper@gmail.com>
# Contributor: Andrea Fagiani <andfagiani _at_ gmail dot com>
# Contributor: Larry Hajali <larryhaja@gmail.com>

pkgbase=calibre
pkgname=('calibre' 'calibre-common' 'calibre-python3')
pkgver=4.4.0
pkgrel=1
pkgdesc="Ebook management application"
arch=('x86_64')
url="https://calibre-ebook.com/"
license=('GPL3')
_py_deps=('apsw' 'beautifulsoup4' 'cssselect' 'css-parser' 'dateutil' 'dbus' 'dnspython'
          'feedparser' 'html2text' 'html5-parser' 'lxml' 'markdown' 'mechanize' 'msgpack'
          'netifaces' 'unrardll' 'pillow' 'psutil' 'pygments' 'pyqt5' 'pyqtwebengine' 'regex')
_py3_deps=("${_py_deps[@]}" 'zeroconf')
depends=('chmlib' 'hunspell' 'icu' 'jxrlib' 'libmtp' 'libusbx' 'libwmf' 'mathjax2' 'mtdev' 'optipng'
         'podofo' 'qt5-svg' 'udisks2')
makedepends=("${_py_deps[@]/#/python2-}" "${_py3_deps[@]/#/python-}" 'qt5-x11extras'
             'rapydscript-ng' 'sip' 'xdg-utils')
checkdepends=('xorg-server-xvfb')
source=("https://download.calibre-ebook.com/${pkgver}/calibre-${pkgver}.tar.xz"
        "https://calibre-ebook.com/signatures/${pkgbase}-${pkgver}.tar.xz.sig"
        "calibre-alternatives.sh")
sha256sums=('089ef0fedda62c0bf6f3fe87fbfe9c87e9537ff985bbd4cb376189326f6a77a2'
            'SKIP'
            '20dc4ff196423a7c7c8f644cb83fcfe07b4b5a64ba4addeb0750f94cd7aa9e8e')
b2sums=('d6f6bfb98dd95012a7d145d7700a2818974b2d2e9f8faf1d98b931283305c92ec7e136c9569ef2074d748cc39819e53186ab250b8331d60253d23fddaa1f8a41'
        'SKIP'
        'c08d9587f9bb5c9b0f4be71bf5189515d2add7932ac5b504a11c8f5130fff129589a0cb58b7d4ffb172f454d993330b01ff153f59cbe4b626afec11f142ed631')
validpgpkeys=('3CE1780F78DD88DF45194FD706BC317B515ACE7C') # Kovid Goyal (New longer key) <kovid@kovidgoyal.net>

prepare(){
    cd "${pkgbase}-${pkgver}"

    # Desktop integration (e.g. enforce arch defaults)
    # Use uppercase naming scheme, don't delete config files under fakeroot.
    sed -e "/import config_dir/,/os.rmdir(config_dir)/d" \
        -e "s/'ctc-posml'/'text' not in mt and 'pdf' not in mt and 'xhtml'/" \
        -e "s/^Name=calibre/Name=Calibre/g" \
        -i  src/calibre/linux.py

    # cherry-picked bits of python2-backports.functools_lru_cache
    # needed for frozen builds + beautifulsoup4
    # see https://github.com/kovidgoyal/calibre/commit/b177f0a1096b4fdabd8772dd9edc66662a69e683#commitcomment-33169700
    rm -r src/backports

    cd resources

    # Remove unneeded files
    rm ${pkgbase}-portable.* mozilla-ca-certs.pem

    # use system mathjax
    rm -r mathjax
}

build() {
    cd "${pkgbase}-${pkgver}"

    LANG='en_US.UTF-8' python2 setup.py build
    LANG='en_US.UTF-8' python2 setup.py gui
    LANG='en_US.UTF-8' python2 setup.py mathjax --path-to-mathjax /usr/share/mathjax2 --system-mathjax
    LANG='en_US.UTF-8' python2 setup.py rapydscript

    LANG='en_US.UTF-8' CALIBRE_PY3_PORT=1 python3 setup.py build
}

check() {
    cd "${pkgbase}-${pkgver}"

    # without xvfb-run this fails with much "Control socket failed to recv(), resetting"
    # ERROR: test_websocket_perf (calibre.srv.tests.web_sockets.WebSocketTest)
    # one or two tests are a bit flaky, but the python3 build seems to succeed more often
    LANG='en_US.UTF-8' xvfb-run env CALIBRE_PY3_PORT=1 python3 setup.py test
    LANG='en_US.UTF-8' xvfb-run python2 setup.py test
}

package_calibre-common() {
    pkgdesc+=" (common files)"
    optdepends=('poppler: required for converting pdf to html')
    conflicts=("calibre<${pkgver}-${pkgrel}")
    install=calibre-common.install
    cd "${pkgbase}-${pkgver}"

    # If this directory doesn't exist, zsh completion won't install.
    install -d "${pkgdir}/usr/share/zsh/site-functions"

    LANG='en_US.UTF-8' python2 setup.py install \
        --staging-root="${pkgdir}/usr" \
        --prefix=/usr

    for bin in "${pkgdir}"/usr/bin/*; do
        ln -sfT "/usr/lib/calibre/bin/${bin##*/}" "${bin}"
    done

    install -Dm755 "${srcdir}"/calibre-alternatives.sh "${pkgdir}"/usr/bin/calibre-alternatives

    cp -a man-pages/ "${pkgdir}/usr/share/man"

    # not needed at runtime
    rm -r "${pkgdir}"/usr/share/calibre/rapydscript/

    #cleanup overlapping files
    rm -r "${pkgdir}"/usr/lib/python2.7
    rm -r "${pkgdir}"/usr/lib/calibre/calibre/plugins/
    find "${pkgdir}" -type f -name '*.py[co]' -delete
}

package_calibre() {
    pkgdesc+=" (python2 build)"
    depends=('calibre-common' "${_py_deps[@]/#/python2-}")
    optdepends+=('ipython2: to use calibre-debug')
    install=calibre.install

    cd "${pkgbase}-${pkgver}"

    LANG='en_US.UTF-8' python2 setup.py install \
        --staging-root="${pkgdir}/usr" \
        --prefix=/usr \
        --no-postinstall \
        --bindir=/usr/lib/calibre/bin-py2 \
        --staging-bindir="${pkgdir}/usr/lib/calibre/bin-py2" \
        --staging-sharedir="${srcdir}"/temp

    # Compiling bytecode FS#33392
    # This is kind of ugly but removes traces of the build root.
    while read -rd '' _file; do
        _destdir="$(dirname "${_file#${pkgdir}}")"
        python2 -m compileall -d "${_destdir}" "${_file}"
        python2 -O -m compileall -d "${_destdir}" "${_file}"
    done < <(find "${pkgdir}"/usr/lib/ -name '*.py' -print0)

    # cleanup overlapping files
    find "${pkgdir}"/usr/lib/calibre -name '*.py' -delete
    rm -r "${pkgdir}"/usr/lib/calibre/calibre/plugins/3/
}

package_calibre-python3() {
    pkgdesc+=" (experimental python3 port)"
    depends=('calibre-common' "${_py3_deps[@]/#/python-}")
    optdepends+=('ipython: to use calibre-debug')
    install=calibre.install

    cd "${pkgbase}-${pkgver}"

    LANG='en_US.UTF-8' CALIBRE_PY3_PORT=1 python3 setup.py install \
        --staging-root="${pkgdir}/usr" \
        --prefix=/usr \
        --no-postinstall \
        --bindir=/usr/lib/calibre/bin-py3 \
        --staging-bindir="${pkgdir}/usr/lib/calibre/bin-py3" \
        --staging-sharedir="${srcdir}"/temp

    # Compiling bytecode FS#33392
    # This is kind of ugly but removes traces of the build root.
    while read -rd '' _file; do
        _destdir="$(dirname "${_file#${pkgdir}}")"
        python3 -m compileall -d "${_destdir}" "${_file}"
        python3 -O -m compileall -d "${_destdir}" "${_file}"
    done < <(find "${pkgdir}"/usr/lib/ -name '*.py' -print0)

    # cleanup overlapping files
    find "${pkgdir}"/usr/lib/calibre -name '*.py' -delete
    rm "${pkgdir}"/usr/lib/calibre/calibre/plugins/*.so
}
