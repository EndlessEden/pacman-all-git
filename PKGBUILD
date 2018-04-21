# Maintainer: Eli Schwartz <eschwartz@archlinux.org>
# Contributor: Eden Rose  <eenov1988 (at) gmail.com>
# Contributor: Dave Reisner <d@falconindy.com>
# Contributor: Thomas Dziedzic < gostrc at gmail >
# Contributor: godane <slaxemulator@gmail.com.com>
# Contributor: Andres Perera <aepd87@gmail.com>

pkgname=pacman-all-git
pkgver=5.0.1.249.g27f64e37
pkgrel=1
pkgdesc="A library-based package manager with dependency support. git version."
arch=('i686' 'x86_64' 'armv7h')
url="http://www.archlinux.org/pacman/"
license=('GPL')
depends=('archlinux-keyring' 'bash' 'curl' 'gpgme' 'libarchive'
         'pacman-mirrorlist' 'pacman-contrib-git')
#optdepends=('pacman-contrib: various helper utilities') - Removed 04/20/18 / pacman-contrib should be a depend.
makedepends=('git' 'asciidoc')
checkdepends=('python2' 'fakechroot')
provides=("pacman=${pkgver%.r*}")
conflicts=('pacman')
backup=(etc/pacman.conf
        etc/makepkg.conf)
options=('strip' 'debug')
source=(git+https://git.archlinux.org/pacman.git
        pacman.conf.i686
        pacman.conf.x86_64
        pacman.conf.arm
		makepkg.conf
		makepkg.conf.arm
        0001-Sychronize-filesystem.patch
        0002-Revert-close-stdin-before-running-install-scripts.patch)
sha256sums=('SKIP'
        '0c087d26e80333267391a6e9e34b95a2ffb103cb9391cb53cc5d97ad954af774'
        'c5a3ec55f9d1bc52e5e5b127f76b7b16b79738268691a3e1d842359033e460da'
        '1a493f674d8934bba26867f55e245cb0ed376fa81fe077b1923e66c0b7e74a38'
        '6066d67d818ee36760bf121c76d5019130f7875b3e5ed179b319b810a3a9685b'
	    '382f316406a6ab9723700dd661f9ac2ec2d193265a60fd7358af964c9c1d1429'
	    '12584247cd7190610a25da4927ed7faee9711e480bce9d1bc57a9abb8b533004'
	    'fc075259e1fb65d41616bf7aa55b0cf927eb8999f065a810f9c4b4dcbc25798b')
pkgver() {
  cd pacman
  git describe | sed 's/^v//;s/-/./g'
}

prepare() {
  cd pacman
if [ $(echo $CARCH | grep -c "86") -lt "1" ]; then
  patch -p1 -i ../0001-Sychronize-filesystem.patch
  patch -p1 -i ../0002-Revert-close-stdin-before-running-install-scripts.patch
fi
  ./autogen.sh
}

build() {
  cd "pacman"

  ./configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var \
    --enable-doc \
    --enable-git-version \
    --enable-debug \
    --with-scriptlet-shell=/usr/bin/bash \
    --with-ldconfig=/usr/bin/ldconfig

  make
}

check() {
  make -C "pacman" check
}

package() {
  cd "pacman"

  make DESTDIR="$pkgdir" install

  # set things correctly in the default conf file
  case $CARCH in
    i686)
      mychost="i686-pc-linux-gnu"
      myflags="-march=i686"
      ;;
    x86_64)
      mychost="x86_64-pc-linux-gnu"
      myflags="-march=x86-64"
      ;;
    arm)
      mycarch="arm"
      mychost="armv5tel-unknown-linux-gnueabi"
      myflags="-march=armv5te "
      ;;
    armv6h)
      mycarch="armv6h"
      mychost="armv6l-unknown-linux-gnueabihf"
      myflags="-march=armv6 -mfloat-abi=hard -mfpu=vfp "
      ;;
    armv7h)
      mycarch="armv7h"
      mychost="armv7l-unknown-linux-gnueabihf"
      myflags="-march=armv7-a -mfloat-abi=hard -mfpu=vfpv3-d16 "
      ;;
    aarch64)
      mycarch="aarch64"
      mychost="aarch64-unknown-linux-gnu"
      myflags="-march=armv8-a "
      ;;
  esac
  
    # install Arch specific stuff
  install -dm755 "$pkgdir/etc"
  if [ $(echo $CARCH | grep -c "86") -gt "0" ]; then
   install -m644 "$srcdir/pacman.conf.$CARCH" "$pkgdir/etc/pacman.conf"
   install -m644 "$srcdir/makepkg.conf" "$pkgdir/etc"
  elif [ $(echo $CARCH | grep -c "arm") -gt "0" ]; then
   install -m644 "$srcdir/pacman.conf.arm" "$pkgdir/etc/pacman.conf"
   install -m644 "$srcdir/makepkg.conf.arm" "$pkgdir/etc/makepkg.conf"
  fi 
  
 

  # set things correctly in the default conf files
  sed -i "$pkgdir/etc/pacman.conf" \
    -e "s|@CARCH[@]|$CARCH|g" \
    -e "s|@CHOST[@]|$mychost|g" \
    -e "s|@CARCHFLAGS[@]|$myflags|g"
	
  sed -i "$pkgdir/etc/makepkg.conf" \
    -e "s|@CARCH[@]|$CARCH|g" \
    -e "s|@CHOST[@]|$mychost|g" \
    -e "s|@CARCHFLAGS[@]|$myflags|g"


  # install completion files
  rm -r "$pkgdir/etc/bash_completion.d"
  install -Dm644 scripts/completion/bash_completion "$pkgdir/usr/share/bash-completion/completions/pacman"
  for f in makepkg pacman-key; do
    ln -s pacman "$pkgdir/usr/share/bash-completion/completions/$f"
  done

  install -Dm644 scripts/completion/zsh_completion $pkgdir/usr/share/zsh/site-functions/_pacman
}

# vim: set ts=2 sw=2 et:
