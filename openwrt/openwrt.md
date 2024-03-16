- [prepare](#prepare)
  - [lede](#lede)
  - [lienol](#lienol)
  - [immortalwrt](#immortalwrt)
  - [common](#common)
- [feeds](#feeds)
- [confg](#confg)
- [downlaod](#downlaod)
- [make](#make)
- [uboot](#uboot)
- [ref](#ref)

# prepare

## lede

``` sh
sudo apt update -y
sudo apt full-upgrade -y
sudo apt install -y ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential \
bzip2 ccache cmake cpio curl device-tree-compiler fastjar flex gawk gettext gcc-multilib g++-multilib \
git gperf haveged help2man intltool libc6-dev-i386 libelf-dev libfuse-dev libglib2.0-dev libgmp3-dev \
libltdl-dev libmpc-dev libmpfr-dev libncurses5-dev libncursesw5-dev libpython3-dev libreadline-dev \
libssl-dev libtool lrzsz mkisofs msmtp ninja-build p7zip p7zip-full patch pkgconf python2.7 python3 \
python3-pyelftools python3-setuptools qemu-utils rsync scons squashfs-tools subversion swig texinfo \
uglifyjs upx-ucl unzip vim wget xmlto xxd zlib1g-dev
```

``` sh
git clone https://github.com/coolsnowwolf/lede
```

patch

```
-luci-app-fileassistant，和luci-app-advanced冲突
-libustream-openssl
-luci-app-homeproxy，依赖firewall4，和firewall冲突
```

## lienol

``` sh
sudo apt install -y build-essential asciidoc binutils bzip2 curl gawk gettext git libncurses5-dev libz-dev patch python3.10 python2.7 unzip zlib1g-dev lib32gcc-s1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf libfuse-dev libgsl-dev python3-pyelftools swig python3-dev
```

``` sh
git clone git@github.com:Lienol/openwrt.git
```

patch

```
ls -lh build_dir/target-*_musl/linux-*/linux-*/Makefile
build_dir/target-aarch64_cortex-a53_musl/linux-mediatek_filogic/linux-5.15.150/Makefile
build_dir/target-aarch64_generic_musl/linux-rockchip_armv8/linux-5.15.150/Makefile
build_dir/target-x86_64_musl/linux-x86_64/linux-6.1.78/Makefile
build_dir/target-x86_64_musl/linux-x86_64/linux-5.15.139/Makefile
KBUILD_CFLAGS-$(CONFIG_WERROR) += -Werror

-luci-app-fileassistant，和luci-app-advanced冲突
-Shortcut-FE，编译错误
-verysync，版本太久，下载不到了
```

## immortalwrt

``` sh
sudo apt install -y --fix-missing ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential \
  bzip2 ccache clang cmake cpio curl device-tree-compiler ecj fastjar flex gawk gettext gcc-multilib \
  g++-multilib git gnutls-dev gperf haveged help2man intltool lib32gcc-s1 libc6-dev-i386 libelf-dev \
  libglib2.0-dev libgmp3-dev libltdl-dev libmpc-dev libmpfr-dev libncurses5-dev libncursesw5 \
  libncursesw5-dev libpython3-dev libreadline-dev libssl-dev libtool lld llvm lrzsz mkisofs msmtp \
  nano ninja-build p7zip p7zip-full patch pkgconf python2.7 python3 python3-pip python3-ply \
  python3-docutils python3-pyelftools qemu-utils re2c rsync scons squashfs-tools subversion swig \
  texinfo uglifyjs upx-ucl unzip vim wget xmlto xxd zlib1g-dev
```

``` sh
git clone git@github.com:immortalwrt/immortalwrt.git
```

patch

```
-luci-app-fileassistant，和luci-app-advanced冲突
-libustream-openssl
+SQUID_with-libcap SQUID_with-nettle SQUID_with-libxml2，squid依赖
-luci-app-kcptun
-luci-app-mosdns
-luci-app-socat
```

## common

build_dir/target-x86_64_musl/host
build_dir/target-x86_64_musl/linux-x86_64

feeds/packages/utils/v2dat/Makefile

# feeds

``` sh
sed -i '$a src-git kenzo https://github.com/kenzok8/openwrt-packages' feeds.conf.default
sed -i '$a src-git small https://github.com/kenzok8/small' feeds.conf.default

sed -i '$a src-git opentopd  https://github.com/sirpdboy/sirpdboy-package' feeds.conf.default

sed -i '$a src-git smpackage https://github.com/kenzok8/small-package' feeds.conf.default
```

``` sh
./scripts/feeds clean
./scripts/feeds update -a
./scripts/feeds install -a

./scripts/feeds clean
./scripts/feeds update -a
# rm -rf feeds/luci/applications/luci-app-mosdns
# rm -rf feeds/packages/net/{alist,adguardhome,xray*,v2ray*,v2ray*,sing*,smartdns}
rm -rf feeds/packages/lang/golang
git clone https://github.com/kenzok8/golang feeds/packages/lang/golang
./scripts/feeds install -a 
``` 

# confg

``` sh
make menuconfig
```

# downlaod

``` sh
make download -j8
```

# make

``` sh
make V=s -j1
make V=s -j4
```

# uboot

ax3000t

https://github.com/hanwckf/bl-mt798x

https://cmi.hanwckf.top/p/mt798x-uboot-usage/

SOC=mt7981 BOARD=ax3000t MULTI_LAYOUT=1 ./build.sh

mt7981_ax3000t-fip-fixed-parts-multi-layout.bin

https://forum.openwrt.org/t/openwrt-support-for-xiaomi-ax3000t/180490

https://openwrt.org/inbox/toh/xiaomi/ax3000t

https://github.com/openwrt/openwrt/pull/14054

# ref

https://github.com/openwrt/openwrt/tree/openwrt-23.05

https://github.com/coolsnowwolf/lede

https://github.com/Lienol/openwrt/tree/23.05

https://github.com/immortalwrt/immortalwrt/tree/openwrt-23.05

https://github.com/kenzok8/openwrt-packages

https://github.com/kenzok8/small

https://github.com/kenzok8/small-package

https://github.com/kiddin9/openwrt-packages

https://github.com/immortalwrt/immortalwrt/discussions/1109

https://blog.xiaofuma.top/archives/FzkXCK33

https://www.bilibili.com/read/cv16071219/

