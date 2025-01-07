# Maintainer: Randall Winkhart <idgr@tutanota.com>

# DO NOT DISTRIBUTE THE RESULTING PACKAGE.
# THIS APKBUILD CREATES A "portable" PACKAGE,
# MEANING IT CONTAINS SEVERAL OTHER PACKAGES WITH
# LICENSES NOT DECLARED IN THIS PACKAGE.
# THIS APKBUILD IS FOR PERSONAL USE ONLY.

pkgname=gns3-server-portable
pkgver=3.0.2
pkgrel=2
psutilver=6.1.1
watchdogver=6.0.0
pkgdesc="GNS3 network simulator (server)"
url="https://github.com/GNS3/gns3-server"
arch="all"
license="GPL-3.0-or-later"
depends="busybox dynamips libcap ubridge vpcs python3 qemu-system-x86_64 qemu-img"
makedepends="py3-pip py3-build py3-wheel python3-dev twine linux-headers"
options="!tracedeps"
install="$pkgname.pre-install"
source="gns3-server-$pkgver.tar.gz::https://github.com/GNS3/gns3-server/archive/v$pkgver.tar.gz
psutil-$psutilver.tar.gz::https://files.pythonhosted.org/packages/1f/5a/07871137bb752428aa4b659f910b399ba6f291156bdea939be3e96cae7cb/psutil-$psutilver.tar.gz
watchdog-$watchdogver.tar.gz::https://files.pythonhosted.org/packages/db/7d/7f3d619e951c88ed75c6037b246ddcf2d322812ee8ea189be89511721d54/watchdog-$watchdogver.tar.gz"

build() {
    cd "gns3-server-$pkgver"
    python3 -m build --wheel
    cd "../psutil-$psutilver"
    python3 -m build --wheel
    cd "../watchdog-$watchdogver"
    python3 -m build --wheel
}

check() {
    cd "$srcdir"/gns3-server-"$pkgver"
    twine check dist/*
    cd "../psutil-$psutilver"
    twine check dist/*
    cd "../watchdog-$watchdogver"
    twine check dist/*
}

package() {
    # install gns3server to $pkgdir
    cd "$srcdir"/gns3-server-"$pkgver"
    wheel unpack ./dist/gns3_server-"$pkgver"-py3-none-any.whl -d "$pkgdir"/usr/lib/
    mv "$pkgdir/usr/lib/gns3_server-$pkgver" "$pkgdir/usr/lib/gns3server"
    printf "#!/usr/bin/python3\n# -*- coding: utf-8 -*-\nimport re\nimport sys\nfrom gns3server.main import main\nif __name__ == '__main__':\n    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])\n    sys.exit(main())" | install -Dm 0755 /dev/stdin "$pkgdir/usr/lib/gns3server/gns3server-bin"
    printf "#!/usr/bin/python3\n# -*- coding: utf-8 -*-\nimport re\nimport sys\nfrom gns3server.utils.vmnet import main\nif __name__ == '__main__':\n    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])\n    sys.exit(main())" | install -Dm 0755 /dev/stdin "$pkgdir/usr/lib/gns3server/gns3vmnet"
    mkdir -p "$pkgdir"/usr/bin
    ln -s "/usr/lib/gns3server/gns3server-bin" "$pkgdir/usr/bin/gns3server"
    ln -s "/usr/lib/gns3server/gns3vmnet" "$pkgdir/usr/bin/gns3vmnet"

    # remove psutil and watchdog from requirements.txt (they are installed separately)
    sed -i '/^psutil/d' requirements.txt
    sed -i '/^watchdog/d' requirements.txt

    # remove sentry-sdk from requirements.txt (it is optional)
    sed -i '/^sentry-sdk/d' requirements.txt

    # download and install pip dependencies into portable directory
    pip download -r ./requirements.txt -d ./localTemp
    for whl in ./localTemp/*.whl; do
            wheel unpack "$whl" -d ./localTemp/unpacked
    done
    for unpacked in ./localTemp/unpacked/*; do
            mv "$unpacked"/* "$pkgdir/usr/lib/gns3server/"
    done

    # install psutil into portable directory
    cd "$srcdir/psutil-$psutilver"
    wheel unpack ./dist/psutil-"$psutilver"-cp36-abi3-linux_x86_64.whl -d "$pkgdir"/usr/lib/gns3server
    mv "$pkgdir/usr/lib/gns3server/psutil-$psutilver/psutil" "$pkgdir"/usr/lib/gns3server/
    mv "$pkgdir/usr/lib/gns3server/psutil-$psutilver/psutil-$psutilver.dist-info" "$pkgdir"/usr/lib/gns3server/
    rm -rf "$pkgdir/usr/lib/gns3server/psutil-$psutilver"

    # install watchdog into portable directory
    cd "$srcdir/watchdog-$watchdogver"
    wheel unpack ./dist/watchdog-"$watchdogver"-py3-none-any.whl -d "$pkgdir"/usr/lib/gns3server
    mv "$pkgdir/usr/lib/gns3server/watchdog-$watchdogver/watchdog" "$pkgdir"/usr/lib/gns3server/
    mv "$pkgdir/usr/lib/gns3server/watchdog-$watchdogver/watchdog-$watchdogver.dist-info" "$pkgdir"/usr/lib/gns3server/
    rm -rf "$pkgdir/usr/lib/gns3server/watchdog-$watchdogver"

    # install OpenRC service
    printf "#!/sbin/openrc-run\nsupervisor=supervise-daemon\n\nname="gns3"\n\ncommand="/usr/bin/gns3server"\ncommand_user=gns3:gns3\n\ndepend() {\n    need net localmount\n    after firewall\n}\n" | install -Dm 0755 /dev/stdin "$pkgdir/etc/init.d/gns3"
}

sha512sums="
a0bf3fb56357adb3b3ef7c7e272777fac017846f7e680c764834b53595af449e0da9bef0e8b8b6e4fa73d078f2a23e07ae967d7b7f3c3ff9a4feb217567af39a  gns3-server-$pkgver.tar.gz
db8a2f4b0b451ca46aaa21b1faae03c4328b1effd04f240a7c8efc94a1c8ca7fc080fc6d16f6ca2046b9232ec43e447be0c414b125f8f511131dc6dff95bd72c  psutil-$psutilver.tar.gz
a248e05b538a894a00e38dabbc0cc7490bb1a87689aa14aac5835f611136fcf6f1e3bd45a6ca4f10991ffabd9264c72f981f4619bd77eb8baf297d015975d57f  watchdog-$watchdogver.tar.gz
"
