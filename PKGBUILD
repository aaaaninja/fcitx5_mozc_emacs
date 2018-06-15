# $Id$
# Maintainer: Jiachen Yang <farseerfc@archlinux.org>
# Contributor: Felix Yan <felixonmars@archlinux.org>
# Contributor: ponsfoot <cabezon dot hashimoto at gmail dot com>
# Contributor: UTUMI Hirosi <utuhiro78 at yahoo dot co dot jp>

## Mozc compile option
_bldtype=Debug

_mozc_rev=280e38fe3d9db4df52f0713acf2ca65898cd697a
_japanese_usage_dictionary_rev=e5b3425575734c323e1d947009dd74709437b684
_gyp_rev=920ee58c3d3109dea3cd37d88054014891a93db7
_protobuf_rev=a428e42072765993ff674fda72863c9f1aa2d268
_zipcode_rel=201805

_pkgbase=mozc
pkgname=fcitx5-mozc-git
pkgdesc="Fcitx5 Module of A Japanese Input Method for Chromium OS, Windows, Mac and Linux (the Open Source Edition of Google Japanese Input)"
pkgver=2.18.2612.102.1.r1316.ffb9ffb0
_fcitx_patchver=2.18.2612.102.1
pkgrel=1
arch=('x86_64')
url="https://github.com/google/mozc"
license=('custom')
depends=('qt5-base' 'fcitx5-git' 'zinnia')
makedepends=('pkg-config' 'python2' 'curl' 'gtk2' 'mesa' 'subversion' 'ninja' 'git' 'clang')
replaces=('mozc-fcitx')
conflicts=('mozc' 'mozc-server' 'mozc-utils-gui' 'mozc-fcitx' 'fcitx-mozc')
source=(git+https://gitlab.com/fcitx/mozc.git#branch=fcitx
        zero_query_dict-iterator-decrement.patch
        #japanese_usage_dictionary::git+https://github.com/hiroyuki-komatsu/japanese-usage-dictionary.git#commit=${_japanese_usage_dictionary_rev}
        #mozc-gyp::git+https://chromium.googlesource.com/external/gyp#commit=${_gyp_rev}
        #git+https://github.com/google/protobuf.git#commit=${_protobuf_rev}
        https://downloads.sourceforge.net/pnsft-aur/x-ken-all-${_zipcode_rel}.zip
        https://downloads.sourceforge.net/pnsft-aur/jigyosyo-${_zipcode_rel}.zip
        https://download.fcitx-im.org/fcitx-mozc/fcitx-mozc-icon.tar.gz
        git+https://chromium.googlesource.com/breakpad/breakpad
        git+https://github.com/google/googletest.git
        git+https://chromium.googlesource.com/external/gyp
        git+https://github.com/hiroyuki-komatsu/japanese-usage-dictionary.git
        git+https://github.com/open-source-parsers/jsoncpp.git
        git+https://github.com/google/protobuf.git
        git+https://github.com/taku910/zinnia.git
	)
sha512sums=('SKIP'
            '9284b6865ee063a294369f40947a2ff7fc3ce49a2bbe9ebbf282a0e0bf199cedc18e1bc51ba8c3e4ed00404f8ef36e6a9db0602b51083f358a1ff555fd031858'
            'e218ae353d2c6f6c9b7c6a0a39398d215fb6218b7842838f137aae793a3d8ec20737b2c82f82f3e8646f5ffce2f052537d22481d958a6eb997f88dc4c8aeca46'
            '4f1274c4acf4f30f72b5bd1827e4dda253fd97951217c15517954b211c08c6b2280c452c1e82e3e9cbf85ac60df96f620a5f7b3a097696d35eeb037ec1b6dd46'
            '5507c637e5a65c44ccf6e32118b6d16647ece865171b9a77dd3c78e6790fbd97e6b219e68d2e27750e22074eb536bccf8d553c295d939066b72994b86b2f251a'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP')
validpgpkeys=('2CC8A0609AD2A479C65B6D5C8E8B898CBF2412F9')  # Weng Xuetian

pkgver() {
  cd mozc
  printf "%s.r%s.%s" "${_fcitx_patchver}" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
  cd "$srcdir/mozc"
  git submodule init
  git config submodule.src/third_party/breakpad.url "$srcdir/breakpad"
  git config submodule.src/third_party/gtest.url "$srcdir/googletest"
  git config submodule.src/third_party/gyp.url "$srcdir/gyp"
  git config submodule.src/third_party/japanese_usage_dictionary.url "$srcdir/japanese-usage-dictionary"
  git config submodule.src/third_party/jsoncpp.url "$srcdir/jsoncpp"
  git config submodule.src/third_party/protobuf.url "$srcdir/protobuf"
  git config submodule.src/third_party/zinnia.url "$srcdir/zinnia"
  git submodule update


  ## Apply fcitx patch
  #rm unix/fcitx -rf
  #patch -Np2 -i "$srcdir/fcitx-mozc-${_fcitx_patchver}.patch"
  patch -Np1 -i "$srcdir/zero_query_dict-iterator-decrement.patch"

  # Adjust to use python2
  find . -name  \*.py        -type f -exec sed -i -e "1s|python.*$|python2|"  {} +
  find scripts               -type f -exec sed -i -e "s|python |python2 |"  {} +
  find . -regex '.*\.gypi?$' -type f -exec sed -i -e "s|'python'|'python2'|g" {} +

  cd src
  # Generate zip code seed
  msg "Generating zip code seed..."
  PYTHONPATH="$PWD:$PYTHONPATH" python2 dictionary/gen_zip_code_seed.py --zip_code="${srcdir}/x-ken-all.csv" --jigyosyo="${srcdir}/JIGYOSYO.CSV" >> data/dictionary_oss/dictionary09.txt
  msg "Done."

  # disable fcitx4 target 
  rm unix/fcitx/fcitx.gyp

  ## Copy third party deps
  #cd "$srcdir"
  #for dep in gyp protobuf japanese_usage_dictionary
  #do
  #  cp -a $dep mozc/src/third_party/
  #done
}

build() {
  # Fix compatibility with google-glog 0.3.3 (symbol conflict)
  CFLAGS="${CFLAGS} -fvisibility=hidden"
  CXXFLAGS="${CXXFLAGS} -fvisibility=hidden"

  cd mozc/src

  _targets="server/server.gyp:mozc_server gui/gui.gyp:mozc_tool unix/fcitx5/fcitx5.gyp:fcitx5-mozc"

  QTDIR=/usr GYP_DEFINES="document_dir=/usr/share/licenses/$pkgname use_libzinnia=1" python2 build_mozc.py gyp
  python2 build_mozc.py build -c $_bldtype $_targets

  #cd mozc/src
  #../scripts/configure
  #../scripts/build_fcitx5

  # Extract license part of mozc
  head -n 29 server/mozc_server.cc > LICENSE
}

package() {
  #cd mozc/src
  #install -D -m 755 out_linux/${_bldtype}/mozc_server "${pkgdir}/usr/lib/mozc/mozc_server"
  #install    -m 755 out_linux/${_bldtype}/mozc_tool   "${pkgdir}/usr/lib/mozc/mozc_tool"

  cd mozc/src
  export PREFIX="${pkgdir}/usr"
  ../scripts/install_server

  install -d "${pkgdir}/usr/share/licenses/$pkgname/"
  install -m 644 LICENSE data/installer/*.html "${pkgdir}/usr/share/licenses/${pkgname}/"

  #for mofile in out_linux/${_bldtype}/gen/unix/fcitx/po/*.mo
  #do
  #  filename=`basename $mofile`
  #  lang=${filename/.mo/}
  #  install -D -m 644 "$mofile" "${pkgdir}/usr/share/locale/$lang/LC_MESSAGES/fcitx-mozc.mo"
  #done

  #install -D -m 755 out_linux/${_bldtype}/fcitx-mozc.so "${pkgdir}/usr/lib/fcitx/fcitx-mozc.so"
  #install -D -m 644 unix/fcitx/fcitx-mozc.conf "${pkgdir}/usr/share/fcitx/addon/fcitx-mozc.conf"
  #install -D -m 644 unix/fcitx/mozc.conf "${pkgdir}/usr/share/fcitx/inputmethod/mozc.conf"

  #install -d "${pkgdir}/usr/share/fcitx/mozc/icon"
  #install -m 644 "$srcdir/fcitx-mozc-icons/mozc.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc.png"
  #install -m 644 "$srcdir/fcitx-mozc-icons/mozc-alpha_full.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-alpha_full.png"
  #install -m 644 "$srcdir/fcitx-mozc-icons/mozc-alpha_half.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-alpha_half.png"
  #install -m 644 "$srcdir/fcitx-mozc-icons/mozc-direct.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-direct.png"
  #install -m 644 "$srcdir/fcitx-mozc-icons/mozc-hiragana.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-hiragana.png"
  #install -m 644 "$srcdir/fcitx-mozc-icons/mozc-katakana_full.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-katakana_full.png"
  #install -m 644 "$srcdir/fcitx-mozc-icons/mozc-katakana_half.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-katakana_half.png"
  #install -m 644 "$srcdir/fcitx-mozc-icons/mozc-dictionary.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-dictionary.png"
  #install -m 644 "$srcdir/fcitx-mozc-icons/mozc-properties.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-properties.png"
  #install -m 644 "$srcdir/fcitx-mozc-icons/mozc-tool.png" "${pkgdir}/usr/share/fcitx/mozc/icon/mozc-tool.png"
  install -d "${PREFIX}/share/fcitx5/addon"
  install -d "${PREFIX}/share/fcitx5/inputmethod"
  install -d "${PREFIX}/lib/fcitx5"
  ../scripts/install_fcitx5

  install -d "${pkgdir}/usr/share/fcitx5/mozc/icon"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc.png" "${pkgdir}/usr/share/fcitx5/mozc/icon/mozc.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-alpha_full.png" "${pkgdir}/usr/share/fcitx5/mozc/icon/mozc-alpha_full.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-alpha_half.png" "${pkgdir}/usr/share/fcitx5/mozc/icon/mozc-alpha_half.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-direct.png" "${pkgdir}/usr/share/fcitx5/mozc/icon/mozc-direct.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-hiragana.png" "${pkgdir}/usr/share/fcitx5/mozc/icon/mozc-hiragana.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-katakana_full.png" "${pkgdir}/usr/share/fcitx5/mozc/icon/mozc-katakana_full.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-katakana_half.png" "${pkgdir}/usr/share/fcitx5/mozc/icon/mozc-katakana_half.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-dictionary.png" "${pkgdir}/usr/share/fcitx5/mozc/icon/mozc-dictionary.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-properties.png" "${pkgdir}/usr/share/fcitx5/mozc/icon/mozc-properties.png"
  install -m 644 "$srcdir/fcitx-mozc-icons/mozc-tool.png" "${pkgdir}/usr/share/fcitx5/mozc/icon/mozc-tool.png"

}
