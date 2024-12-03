# Maintainer: Randall Winkhart <idgr@tutanota.com>

# DO NOT DISTRIBUTE THE RESULTING PACKAGE.
# THIS APKBUILD CREATES A "portable" PACKAGE,
# MEANING IT CONTAINS SEVERAL OTHER PACKAGES WITH
# LICENSES NOT DECLARED IN THIS PACKAGE.
# THIS APKBUILD IS FOR PERSONAL USE ONLY.

pkgname=gns3-server-portable
pkgver=2.2.52
pkgrel=0
psutilver=6.1.0
pkgdesc="GNS3 network simulator (server)"
url="https://github.com/GNS3/gns3-server"
arch="all"
license="GPL-3.0-or-later"
depends="busybox dynamips libcap ubridge vpcs python3 qemu-system-x86_64 qemu-img"
makedepends="py3-pip py3-setuptools py3-wheel linux-headers python3-dev"
options="!tracedeps"
install="$pkgname.pre-install"
source="gns3-server-$pkgver.tar.gz::https://github.com/GNS3/gns3-server/archive/v$pkgver.tar.gz
psutil-$psutilver.tar.gz::https://files.pythonhosted.org/packages/26/10/2a30b13c61e7cf937f4adf90710776b7918ed0a9c434e2c38224732af310/psutil-$psutilver.tar.gz"

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
15c13a9c3ebb1b0622fee512bedce2ed77eb89b47b03f2eac81c37ec1bbf5edffa34b6fe149d61e889c2b32c752a497888fa0e8288579d24d5d5c642513ecf48  gns3-server-$pkgver.tar.gz
76865df4fdb2a9df45e47589b76b34d0d9d9251491091683e47b4509863e32e46dc62ee2f760b983f0f762b8288d1ea7f32268a6857c049ad12f399908e19c82  psutil-$psutilver.tar.gz
"
