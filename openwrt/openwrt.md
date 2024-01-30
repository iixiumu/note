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

## lienol

``` sh
sudo apt install -y build-essential asciidoc binutils bzip2 curl gawk gettext git libncurses5-dev libz-dev patch python3.10 python2.7 unzip zlib1g-dev lib32gcc-s1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf
```

``` sh
git clone git@github.com:Lienol/openwrt.git
git checkout 23.05
```

# feeds

``` sh
sed -i '$a src-git kenzo https://github.com/kenzok8/openwrt-packages' feeds.conf.default
sed -i '$a src-git small https://github.com/kenzok8/small' feeds.conf.default
```

``` sh
./scripts/feeds clean
./scripts/feeds update -a
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



