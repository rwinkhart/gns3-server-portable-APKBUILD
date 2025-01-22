# Maintainer: Randall Winkhart <idgr@tutanota.com>

# DO NOT DISTRIBUTE THE RESULTING PACKAGE.
# THIS APKBUILD CREATES A "portable" PACKAGE,
# MEANING IT CONTAINS SEVERAL OTHER PACKAGES WITH
# LICENSES NOT DECLARED IN THIS PACKAGE.
# THIS APKBUILD IS FOR PERSONAL USE ONLY.

pkgname=gns3-server-portable
pkgver=2.2.53
pkgrel=0
psutilver=6.1.1
pkgdesc="GNS3 network simulator (server)"
url="https://github.com/GNS3/gns3-server"
arch="all"
license="GPL-3.0-or-later"
depends="busybox dynamips libcap ubridge vpcs python3 qemu-system-x86_64 qemu-img"
makedepends="py3-pip py3-setuptools py3-wheel linux-headers python3-dev"
options="!tracedeps"
install="$pkgname.pre-install"
source="gns3-server-$pkgver.tar.gz::https://github.com/GNS3/gns3-server/archive/v$pkgver.tar.gz
psutil-$psutilver.tar.gz::https://files.pythonhosted.org/packages/1f/5a/07871137bb752428aa4b659f910b399ba6f291156bdea939be3e96cae7cb/psutil-6.1.1.tar.gz"

build() {
	cd "gns3-server-$pkgver"
	python3 setup.py build
	cd "../psutil-$psutilver"
	python3 setup.py build
}

check() {
	cd "$srcdir"/gns3-server-"$pkgver"
	python3 setup.py check
	cd "../psutil-$psutilver"
	python3 setup.py check
}

package() {
	# fetch system python version
	pyver=$(python3 --version | cut -d. -f2)

	# install gns3server to $pkgdir
	cd "$srcdir"/gns3-server-"$pkgver"
	python3 setup.py install --prefix=/usr --root="$pkgdir"

	cd "../psutil-$psutilver"
	python3 setup.py install --skip-build --root=psutilTemp
	cd "../gns3-server-$pkgver"

	# re-arrange $pkgdir for portability
	mv "$pkgdir/usr/lib/python3.$pyver/site-packages" "$pkgdir/usr/lib/gns3server"
	mv "$pkgdir/usr/bin/gns3server" "$pkgdir/usr/lib/gns3server/gns3server-bin"
	mv "$pkgdir/usr/bin/gns3loopback" "$pkgdir/usr/lib/gns3server/gns3loopback"
	mv "$pkgdir/usr/bin/gns3vmnet" "$pkgdir/usr/lib/gns3server/gns3vmnet"
	rm -r "$pkgdir/usr/lib/python3.$pyver"
	ln -s "/usr/lib/gns3server/gns3server-bin" "$pkgdir/usr/bin/gns3server"
	ln -s "/usr/lib/gns3server/gns3loopback" "$pkgdir/usr/bin/gns3loopback"
	ln -s "/usr/lib/gns3server/gns3vmnet" "$pkgdir/usr/bin/gns3vmnet"
	
	# remove psutil from requirements.txt (it is installed separately)
	sed -i '/^psutil/d' requirements.txt
	
	# remove sentry-sdk from requirements.txt (it is optional)
	sed -i '/^sentry-sdk/d' requirements.txt

	# download and install pip dependencies into portable directory
	pip download -r ./requirements.txt -d ./localTemp
	for whl in ./localTemp/*.whl; do
        	wheel unpack "$whl" -d ./localTemp/unpacked
	done
	for package in ./localTemp/unpacked/*/*/; do
        	mv "$package" "$pkgdir/usr/lib/gns3server/"
	done

	# install psutil into portable directory
	mv "../psutil-$psutilver/psutilTemp/usr/lib/python3.$pyver/site-packages/psutil" "$pkgdir/usr/lib/gns3server/psutil"
	mv "../psutil-$psutilver/psutilTemp/usr/lib/python3.$pyver/site-packages/psutil-$psutilver-py3.$pyver.egg-info" "$pkgdir/usr/lib/gns3server/psutil-$psutilver-py3.$pyver.egg-info"

	# install OpenRC service
	printf "#!/sbin/openrc-run\nsupervisor=supervise-daemon\n\nname="gns3"\n\ncommand="/usr/bin/gns3server"\ncommand_user=gns3:gns3\n\ndepend() {\n    need net localmount\n    after firewall\n}\n" | install -Dm 0755 /dev/stdin "$pkgdir/etc/init.d/gns3"
}

sha512sums="
d9c8d08bab3cf9b246c90e7b35a7ff8fd9dcb29b4b16402a8bf0bd0013ff14a88ac4f0f4b0a185d257ffc86ff744cf1ef73112abe21378e3282c0b10464602d9  gns3-server-$pkgver.tar.gz
db8a2f4b0b451ca46aaa21b1faae03c4328b1effd04f240a7c8efc94a1c8ca7fc080fc6d16f6ca2046b9232ec43e447be0c414b125f8f511131dc6dff95bd72c  psutil-$psutilver.tar.gz
"
