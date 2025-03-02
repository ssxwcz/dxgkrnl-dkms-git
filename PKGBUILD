# Maintainer: ssxw <ssxwcz@gmail.com>
pkgname=dxgkrnl-dkms-git
pkgver=1.0
pkgrel=1
pkgdesc="Microsoft dxgkrnl kernel module for GPU support in the latest self-compiled WSL2 Linux kernel"
arch=('x86_64')
url="https://github.com/ssxwcz/dxgkrnl-dkms-git"
license=('GPL2')
depends=('dkms')
makedepends=('git' 'linux-headers')
provides=("dxgkrnl")
source=('0001-Add-a-gpu-pv-support.patch' '0002-Fix-eventfd_signal.patch')
sha256sums=('9994e8b021341f7805f5e51a38cd065abaff8f106bceefc3138bd859be48865b' '44c24dbbea7833689cae5395e6f991124826e547172c8a41c177018687063252' )

prepare() {

    # 清理缓存文件以及拉取 dxgkrnl 驱动源码
    # 详见 https://github.com/Nevuly/WSL2-Linux-Kernel-Rolling-LTS/tree/wsl-6.12-lts/drivers/hv/dxgkrnl
    rm -rf dxgkrnl-*
    rm -rf WSL2-Linux-Kernel-sparse
    git clone --branch=linux-msft-wsl-6.6.y --no-checkout --depth=1 --filter=blob:none https://github.com/microsoft/WSL2-Linux-Kernel.git WSL2-Linux-Kernel-sparse
    cd WSL2-Linux-Kernel-sparse
    git sparse-checkout set --no-cone /drivers/hv/dxgkrnl /include/uapi/misc/d3dkmthk.h
    git checkout -f linux-msft-wsl-6.6.y
    git apply -v $srcdir/0001-Add-a-gpu-pv-support.patch
    git apply -v $srcdir/0002-Fix-eventfd_signal.patch

    commit_version=$(git rev-parse --short HEAD)
    msg "Git commit version: $commit_version"

    # 复制源码和头文件到对应目录
    mkdir -p $srcdir/dxgkrnl-$commit_version
    mkdir -p $srcdir/dxgkrnl-$commit_version/include
    cp -r drivers/hv/dxgkrnl/. $srcdir/dxgkrnl-$commit_version/
    cp -r include/. $srcdir/dxgkrnl-$commit_version/include/

    # 修改 makefile 配置
    sed -i 's/\$(CONFIG_DXGKRNL)/m/' "$srcdir/dxgkrnl-$commit_version/Makefile"
    echo "EXTRA_CFLAGS=-I\$(PWD)/include -D_MAIN_KERNEL_" >> "$srcdir/dxgkrnl-$commit_version/Makefile"

    # 创建 dkms 配置文件
    cat > $srcdir/dxgkrnl-$commit_version/dkms.conf <<EOF
PACKAGE_NAME="dxgkrnl"
PACKAGE_VERSION="$commit_version"
BUILT_MODULE_NAME="dxgkrnl"
DEST_MODULE_LOCATION="/kernel/drivers/hv/dxgkrnl/"
AUTOINSTALL="yes"
EOF

    # 修改部分源码
    sed -i '0,/^#include.*/s//&\n#include <linux\/vmalloc.h>/' $srcdir/dxgkrnl-$commit_version/dxgadapter.c
    sed -i '0,/^#include.*/s//&\n#include <linux\/vmalloc.h>/' $srcdir/dxgkrnl-$commit_version/dxgvmbus.c
    sed -i '0,/^#include.*/s//&\n#include <linux\/vmalloc.h>/' $srcdir/dxgkrnl-$commit_version/hmgr.c
    sed -i '0,/^#include.*/s//&\n#include <linux\/vmalloc.h>/' $srcdir/dxgkrnl-$commit_version/ioctl.c

    export commit_version
}

package() {

    if [[ ! -d "$srcdir/dxgkrnl-$commit_version" ]]; then
        error "Source directory not found: $srcdir/dxgkrnl-$commit_version"
        exit 1
    fi
    install -dm 755 "${pkgdir}"/usr/src
    cp -dr --no-preserve='ownership' "$srcdir/dxgkrnl-$commit_version" "$pkgdir/usr/src/"
}

