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

_destdir=opt
_installdir=installdir
_installpath=$_installdir/$_destdir/$pkgname
_scriptsdir=$_installpath/${pkgname}/install_scripts

options=(!strip !purge)

build() {
    # 创建伪造的 home 目录以防污染主系统
    mkdir -p "$srcdir/fakehome"

    echo ">>> Installing..."
    # 修正：直接运行下载的 bin 文件，prefix 指向 srcdir 下的临时目录
    chmod +x "${pkgname}-${pkgver}-setup.run"
    HOME="$srcdir/fakehome" ./"${pkgname}-${pkgver}-setup.run" \
        --mode unattended \
        --prefix "$srcdir/opt/$pkgname"
}

package() {
    # 1. 准备安装目录
    mkdir -p "$pkgdir/opt"
    cp -ral "$srcdir/opt/$pkgname" "$pkgdir/opt/"

    # 2. 使用 awk 注入动态路径解析逻辑
    # 替换 DIR=`dirname "$0"` 为能追踪软链接物理位置的代码
    _fix_script() {
        awk '
        /^DIR=`dirname/ {
            print "# 路径修正：支持从软链接启动"
            print "REAL_PATH=$(readlink -f \"$0\")"
            print "DIR=$(dirname \"$REAL_PATH\")"
            next
        }
        { print }' "$1" > "$1.tmp" && mv "$1.tmp" "$1"
    }

    _fix_script "$pkgdir/opt/$pkgname/sysconfig_gui.sh"
    _fix_script "$pkgdir/opt/$pkgname/sysconfig_cli.sh"

    # 3. 创建软链接
    mkdir -p "$pkgdir/usr/bin"
    ln -s "/opt/$pkgname/sysconfig_gui.sh" "$pkgdir/usr/bin/$pkgname"
    ln -s "/opt/$pkgname/sysconfig_cli.sh" "$pkgdir/usr/bin/$pkgname-cli"

    # 4. 修正 .desktop 文件
    _desktop_file="$pkgdir/opt/$pkgname/TI sysconfig.desktop"
    if [ -f "$_desktop_file" ]; then
        sed -i "s#$srcdir/opt#/opt#g" "$_desktop_file"
        # 统一使用 /usr/bin 下的链接启动
        sed -i "s#Exec=.*#Exec=/usr/bin/$pkgname#g" "$_desktop_file"
        install -D -m0644 "$_desktop_file" "$pkgdir/usr/share/applications/$pkgname.desktop"
    fi

    # 5. 权限与许可证
    chmod -R 755 "$pkgdir/opt/$pkgname"
    chmod +x "$pkgdir/usr/bin/"*

    install -d "$pkgdir/usr/share/licenses/$pkgname"
    # 自动寻找可能存在的 LICENSE 文件
    find "$pkgdir/opt/$pkgname" -maxdepth 2 \( -name "LICENSE" -o -name "*license*" \) -exec cp {} "$pkgdir/usr/share/licenses/$pkgname/" \;
}
