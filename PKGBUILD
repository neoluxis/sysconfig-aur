# Maintainer: neolux lee<neolux_lee@outlook.com>


pkgname=sysconfig
_semver=1.27.0
_bldver=4565
pkgver=$_semver.$_bldver
pkgrel=1
pkgdesc="Texas Instruments SysConfig Tool"
arch=('x86_64')
url="http://www.ti.com/tool/SYSCONFIG"
license=('custom:TSPA')

depends=('binutils')

# The license file was copy-pasted from the installer's GUI
_archive=SysConfig_${pkgver}_linux
source=(
    ${pkgname}-${pkgver}-setup.run::"https://dr-download.ti.com/software-development/ide-configuration-compiler-or-debugger/MD-nsUM6f7Vvb/${pkgver}/sysconfig-${_semver}_${_bldver}-setup.run"
)

sha256sums=(
    'aaaeed931c5dea9fa4fa135612d773af5724e2916148bb6947ef4adc1b980517'
)

install=$pkgname.install

options=(!strip libtool staticlibs emptydirs !purge !zipman)

_desktop="SysConfig ${_semver}.desktop"

_destdir=opt/ti/${pkgname}
_installdir=installdir
_installpath=$_installdir/$_destdir/$pkgname
_scriptsdir=$_installpath/${pkgname}/install_scripts

options=(!strip !purge)


_destdir="/opt/ti/$pkgname"

build() {
    # 显式创建完整的临时安装路径
    mkdir -p "$srcdir/fakehome"
    mkdir -p "$srcdir$_destdir"

    chmod +x "${pkgname}-${pkgver}-setup.run"

    echo ">>> 正在安装至临时目录..."
    # 关键：确保 --prefix 指向的是一个已经存在的绝对路径
    HOME="$srcdir/fakehome" ./"${pkgname}-${pkgver}-setup.run" \
        --mode unattended \
        --prefix "$srcdir$_destdir"
}

package() {
    # 1. 确认安装源目录存在
    if [ ! -d "$srcdir$_destdir" ]; then
        echo "错误：安装目录未找到！"
        exit 1
    fi

    # 2. 拷贝到 pkgdir
    mkdir -p "$pkgdir/opt/ti"
    cp -ral "$srcdir/opt/ti/$pkgname" "$pkgdir/opt/ti/"

    # 3. 注入动态路径解析逻辑 (使用 awk)
    _fix_script() {
        awk '/^DIR=`dirname/ {
            print "# Path fix: Support running from symlinks"
            print "REAL_PATH=$(readlink -f \"$0\")"
            print "DIR=$(dirname \"$REAL_PATH\")"
            next
        } { print }' "$1" > "$1.tmp" && mv "$1.tmp" "$1"
    }
    _fix_script "$pkgdir$_destdir/sysconfig_gui.sh"
    _fix_script "$pkgdir$_destdir/sysconfig_cli.sh"

    # 4. 创建系统软链接
    mkdir -p "$pkgdir/usr/bin"
    ln -s "$_destdir/sysconfig_gui.sh" "$pkgdir/usr/bin/$pkgname"
    ln -s "$_destdir/sysconfig_cli.sh" "$pkgdir/usr/bin/$pkgname-cli"

    # 5. 【关键：安装并修正 Desktop 文件】
    echo ">>> 正在处理并安装 Desktop 文件..."
    # 自动定位那个带空格的原始文件，例如 "TI sysconfig.desktop"
    _orig_desktop=$(find "$pkgdir$_destdir" -maxdepth 1 -name "*.desktop" -print -quit)

    if [ -n "$_orig_desktop" ]; then
        # A. 将所有构建时的临时绝对路径替换为系统路径 /opt/ti
        sed -i "s#$srcdir/opt/ti#/opt/ti#g" "$_orig_desktop"

        # B. 修正 Exec：让它指向我们包装好的 /usr/bin/sysconfig
        # 原始的 Exec 往往直接调用 nw 二进制，这会导致某些环境下运行失败
        sed -i "s#Exec=.*#Exec=/usr/bin/$pkgname#g" "$_orig_desktop"

        # C. 修正 Path 和 Icon 路径（如果 sed 没改干净）
        sed -i "s#Path=.*#Path=$_destdir#g" "$_orig_desktop"

        # D. 投递到目的地，并重命名为规范的 sysconfig.desktop
        install -D -m0644 "$_orig_desktop" "$pkgdir/usr/share/applications/$pkgname.desktop"

        # 清理掉 /opt/ti/sysconfig 目录下原始的带空格的 desktop 文件，保持目录整洁
        rm "$_orig_desktop"
    else
        echo ">>> 警告：未在安装目录中找到 .desktop 文件！"
    fi

    # 6. 权限与 License
    echo ">>> 正在修正权限..."
    chmod -R 755 "$pkgdir$_destdir"
    chmod -h 755 "$pkgdir/usr/bin/$pkgname" "$pkgdir/usr/bin/$pkgname-cli"

    install -d "$pkgdir/usr/share/licenses/$pkgname"
    find "$pkgdir$_destdir" -maxdepth 2 \( -name "LICENSE" -o -name "*license*" \) -exec cp {} "$pkgdir/usr/share/licenses/$pkgname/" \;
}
