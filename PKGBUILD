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
    # 1. 修正 .desktop 文件中的路径（从 $srcdir 指向系统的 /opt）
    _desktop_file="$srcdir/opt/$pkgname/TI sysconfig.desktop"
    if [ -f "$_desktop_file" ]; then
        sed -i "s#$srcdir/opt# /opt#g" "$_desktop_file"
        install -D -m0644 "$_desktop_file" "$pkgdir/usr/share/applications/$pkgname.desktop"
    fi

    # 2. 将安装好的整个目录移动到 pkgdir
    mkdir -p "$pkgdir/opt"
    cp -ral "$srcdir/opt/$pkgname" "$pkgdir/opt/"

    # 3. 权限修正
    # SysConfig 有时需要在其目录下写入临时文件或更新，保持与 CCS 类似的权限逻辑
    chmod -R 755 "$pkgdir/opt/$pkgname"

    # 4. 创建可执行文件软链接
    mkdir -p "$pkgdir/usr/bin"
    ln -s "/opt/$pkgname/sysconfig_gui.sh" "$pkgdir/usr/bin/sysconfig-gui"
    ln -s "/opt/$pkgname/sysconfig_gui.sh" "$pkgdir/usr/bin/sysconfig"
    ln -s "/opt/$pkgname/sysconfig_cli.sh" "$pkgdir/usr/bin/sysconfig-cli"

    # 5. 处理 License
    # 尝试从安装目录寻找协议文件
    install -d "$pkgdir/usr/share/licenses/$pkgname"
    find "$pkgdir/opt/$pkgname" -name "*license*" -maxdepth 2 -exec cp {} "$pkgdir/usr/share/licenses/$pkgname/" \;
}
