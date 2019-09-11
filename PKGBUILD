# Maintainer: Moses Narrow <moe_narrow@use.startmail.com>
# Maintainer: Rudi [KittyCash] <rudi@skycoinmail.com>
pkgname=skycoin
pkgname1=skycoin
projectname=skycoin
pkgdesc="Skycoin Cryptocurrency Wallet"
pkgver=0.26.0
pkggopath="github.com/${projectname}/${pkgname1}"
pkgrel=3
arch=('any')
url="https://${pkggopath}"
license=()
makedepends=(dep git go gcc)
source=("${url}/archive/v${pkgver}.tar.gz")
sha256sums=('60f7f2a7c33dbe754ffc74b86de7c2a759e246a83953d4d52fb869d1b3fa1ee2')

case "$CARCH" in
x86)      export GOARCH="386" GO386="387" ;;
x86_64)   export GOARCH="amd64" ;;
arm*)     export GOARCH="arm" ;;
armel)    export GOARCH="arm" GOARM="5" ;;
armhf)    export GOARCH="arm" GOARM="6" ;;
armv7)    export GOARCH="arm" GOARM="7" ;;
armv8)    export GOARCH="arm64" ;;
aarch64)  export GOARCH="arm64" ;;
mips)     export GOARCH="mips" ;;
mips64)   export GOARCH="mips64" ;;
mips64el) export GOARCH="mips64le" ;;
mipsel)   export GOARCH="mipsle" ;;
*)        return 1 ;;
	esac

	prepare() {
	# https://wiki.archlinux.org/index.php/Go_package_guidelines
	mkdir -p ${srcdir}/go/src/${pkggopath//${pkgname1}}/${pkgname1} ${srcdir}/go/bin
	ln -rTsf ${srcdir}/${pkgname1}-$pkgver ${srcdir}/go/src/${pkggopath}
	cd ${srcdir}/go/src/${pkggopath}/cmd

	#git submodule --quiet update --init --recursive

	export GOPATH="${srcdir}"/go
	export GOBIN=${GOPATH}/bin
	export PATH=${GOPATH}/bin:${PATH}
	msg2 'installing go dependencies'
	dep init
	dep ensure
	}

build() {
##detect architecture & adjust build args accordingly
##not used currently because makefile is malfunctioning
#case "$CARCH" in
#	arm*) make release-standalone osarch="linux/arm"
#		;;
#  aarch64*) make release-standalone osarch="linux/arm"
#    ;;
#	x86_64) _pkgarch="$pkgoption1"
#	make release-standalone osarch="linux/amd64"
#		;;
#esac

## manually build
export GOROOT=/usr/lib/go
export GOPATH="${srcdir}"/go
export GOBIN=${GOPATH}/bin
export PATH=${GOPATH}/bin:${PATH}
cd ${srcdir}/go/src/${pkggopath}/cmd/
go install \
  -gcflags "all=-trimpath=${GOPATH}" \
  -asmflags "all=-trimpath=${GOPATH}" \
  -v ./...

msg 2 'creating launcher scripts skycoin-wallet & skycoin-wallet-nohup'
mkdir -p ${srcdir}/${pkgname1}-scripts
cd ${srcdir}/${pkgname1}-scripts
echo -e '#!/bin/bash \n #launch skycoin wallet \n export GOBIN=/usr/lib/skycoin/go/bin \n export GOPATH=GOBIN=/usr/lib/skycoin/go \n skycoin -gui-dir=/usr/lib/skycoin/skycoin/src/gui/static/ -launch-browser=true -enable-all-api-sets=true -enable-gui=true -log-level=debug' > ${pkgname1}-wallet
chmod +x ${pkgname1}-wallet
echo -e '#!/bin/bash \n #launch skycoin wallet with nohup \n export GOBIN=/usr/lib/skycoin/go/bin \n export GOPATH=GOBIN=/usr/lib/skycoin/go \n nohup skycoin -gui-dir=/usr/lib/skycoin/skycoin/src/gui/static -launch-browser=true -enable-all-api-sets=true -enable-gui=true -log-level=debug > /dev/null 2>&1 &echo "skycoin wallet has started"' > ${pkgname1}-wallet-nohup
chmod +x ${pkgname1}-wallet-nohup
echo -e '#!/bin/bash \n #launch skycoin daemon \n export GOBIN=/usr/lib/skycoin/go/bin \n export GOPATH=GOBIN=/usr/lib/skycoin/go \n skycoin -gui-dir=/usr/lib/skycoin/skycoin/src/gui/static/ -enable-gui=false -launch-browser=false -log-level=debug -enable-all-api-sets=true' > ${pkgname1}-node
chmod +x ${pkgname1}-node
echo -e '#!/bin/bash \n #halt skycoin \n sudo killall skycoin \n sudo killall cli \n sudo killall cipher-testdata \n sudo killall newcoin \n sudo killall monitor-peers \n echo "skycoin halted"' > ${pkgname1}-halt
chmod +x ${pkgname1}-halt
#
msg 2 'creating system.d .service files'
#these service files point to skywire & skywire-node-miner scripts from above
#the systemd service files included with skywire are wrong for archlinux (debian formatted)
cd ${srcdir}/go
echo -e '[Unit] \n Description=Skycoin Node service \n	After=network.target \n After=network-online.target \n \n	[Service] \n Type=oneshot \n	ExecStart=/usr/bin/skycoin-node \n	RemainAfterExit=yes \n ExecStop=/usr/bin/skycoin-halt \n	TryExec=/usr/bin/skycoin \n \n [Install] \n	WantedBy=multi-user.target ' > ${pkgname1}-node.service
}

package() {
options=(!strip staticlibs)
#create directory trees
mkdir -p ${pkgdir}/usr/bin
mkdir -p ${pkgdir}/usr/lib/${projectname}/go/bin
mkdir -p ${pkgdir}/usr/lib/${projectname}/${pkgname1}/src/gui
#install binaries & symlink to /usr/bin
msg2 'installing binaries'
skybin="${srcdir}"/go/bin
#avoid generic names for binaries
#collect the binaries & install
skybins=$( ls "$skybin")
for i in ${skybins}; do
  install -Dm755 ${srcdir}/go/bin/${i} ${pkgdir}/usr/lib/${projectname}/go/bin/${i}
  ln -rTsf ${pkgdir}/usr/lib/${projectname}/go/bin/${i} ${pkgdir}/usr/bin/${i}
  chmod 755 ${pkgdir}/usr/bin/${i}
done
#install the web dir (UI)
cp -r ${srcdir}/${pkgname1}-$pkgver/src/gui/static ${pkgdir}/usr/lib/${projectname}/${pkgname1}/src/gui
#install the scripts
skycoinscripts=$( ls ${srcdir}/${pkgname1}-scripts )
for i in ${skycoinscripts}; do
cp ${srcdir}/${pkgname1}-scripts/${i} ${pkgdir}/usr/bin/${i}
done
#correct symlink names
namechange=$(ls --ignore='skycoin*' ${pkgdir}/usr/bin/)
for i in ${namechange}; do
mv ${pkgdir}/usr/bin/${i} ${pkgdir}/usr/bin/${pkgname1}-${i}
done
#install the system.d service
install -Dm644 ${srcdir}/go/${pkgname1}-node.service ${pkgdir}/usr/lib/systemd/system/${pkgname1}-node.service
}
