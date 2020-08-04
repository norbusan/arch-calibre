# Maintainer: Jelle van der Waa <jelle@vdwaa.nl>
# Maintainer: Eli Schwartz <eschwartz@archlinux.org>
# Contributor: Daniel Wallace <danielwallace at gtmanfred dot com>
# Contributor: Giovanni Scafora <giovanni@archlinux.org>
# Contributor: Petrov Roman <nwhisper@gmail.com>
# Contributor: Andrea Fagiani <andfagiani _at_ gmail dot com>
# Contributor: Larry Hajali <larryhaja@gmail.com>

pkgbase=calibre
pkgname=('calibre' 'calibre-common' 'calibre-python3')
pkgver=4.22.0
pkgrel=1
pkgdesc="Ebook management application"
arch=('x86_64')
url="https://calibre-ebook.com/"
license=('GPL3')
_py_deps=('apsw' 'beautifulsoup4' 'cssselect' 'css-parser' 'dateutil' 'dbus' 'dnspython'
          'feedparser' 'html2text' 'html5-parser' 'lxml' 'markdown' 'mechanize' 'msgpack'
          'netifaces' 'unrardll' 'pillow' 'psutil' 'pychm' 'pygments' 'pyqt5'
          'pyqtwebengine' 'regex')
_py2_deps=("${_py_deps[@]}" 'ipaddress')
_py3_deps=("${_py_deps[@]}" 'zeroconf')
depends=('hunspell' 'hyphen' 'icu' 'jxrlib' 'libmtp' 'libusbx'
         'libwmf' 'mathjax' 'mtdev' 'optipng' 'podofo' 'qt5-svg' 'udisks2')
makedepends=("${_py2_deps[@]/#/python2-}" "${_py3_deps[@]/#/python-}" 'qt5-x11extras'
             'rapydscript-ng' 'sip' 'xdg-utils')
checkdepends=('xorg-server-xvfb')
source=("https://download.calibre-ebook.com/${pkgver}/calibre-${pkgver}.tar.xz"
        "https://calibre-ebook.com/signatures/${pkgbase}-${pkgver}.tar.xz.sig"
        "0001-De-vendor-pychm.patch"
        "calibre-alternatives.sh")
sha256sums=('51f52b7015c4d9b456d0d5e8efacfffe564709552f0a635ae8174d1947ab1c34'
            'SKIP'
            'f7b829aea1d33818808cbeeb9a295e18e49edf619a5bc89b8315c88f56ce4d25'
            '940cc7081d0a64ba363bb0e1a1d8e0563c676458f90db845f2fbdd4195c075b3')
b2sums=('19576d5cfc1a4ed6a505ef46656675980b6736be01f55874951a9a0c81a70c82e23e723db1d81d13917eaf615e65752a100fbc1cb43bdca0b3c4543e3b17cf43'
        'SKIP'
        'c35181c70084813772c4d593311b48b3e3bcc3b4e9e8ee58112b9beab2bbc0de1ee22aafc3d06cfd812f87a2e91292f7b7f1dc5f522c55440f415b6b265d5671'
        '543df218dfd2d4152a941ab57118d69bf4c6927e8020ee53c9a8b38efe9c89f032dc6385207e134cc9f69bfdc9cbcf63cd92fa6ea1647cbd534c5a511a5d1e91')
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
    # biplist is only used on macOS + python2
    rm -r src/biplist/
    # devendor pychm now, from the py3 building branch:
    # https://github.com/kovidgoyal/calibre/commit/959b7e3fafff5faad6ae59263f825b23c7563dd4
    patch -p1 -i ../0001-De-vendor-pychm.patch

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
    LANG='en_US.UTF-8' python2 setup.py mathjax --path-to-mathjax /usr/share/mathjax --system-mathjax
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
    depends=('calibre-common' "${_py2_deps[@]/#/python2-}")
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
