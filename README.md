# Portable/Bundled GNS3-server for Alpine Linux
> [!WARNING]
> I strongly advise against redistributing the package resulting from this APKBUILD.
> The resulting package is a bundle of various pieces of software with different licenses.
> Because of this, the only way to ensure no license is being violated is to use this
> APKBUILD and the resulting package for **personal use only**.

This APKBUILD can be used to generate a .apk package for Alpine Linux containing GNS3-server with all of its Python dependencies bundled in (using the versions recommended by upstream GNS3-server).

The package also creates a gns3 user and group to run the server under.

The server can be easily managed using an included OpenRC init script.

# How-To
Ensure your Alpine Linux system is configured to build packages using abuild.
If you have not done this already, a basic setup can be achieved using the following:
```
doas apk add alpine-sdk
doas addgroup <user> abuild
# re-log or reboot
abuild-keygen -ai
```

Now you may use the contents of this repo to generate the desired package:
```
git clone https://github.com/rwinkhart/gns3-server-portable-APKBUILD.git
cd gns3-server-portable-APKBUILD
abuild -r
```

The resulting package will be placed at `~/packages/home/x86_64/gns3-server-portable-<version>.apk`

It can be installed with `doas apk add </path/to/.apk>`.

If installing on a system not configured to trust your abuild key, you will need to specify `--allow-untrusted` at the end of the above installation command.
