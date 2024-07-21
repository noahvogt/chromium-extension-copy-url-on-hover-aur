# Maintainer: Noah Vogt (noahvogt) <noah@noahvogt.com>
# private key generated with `openssl genrsa 2048 | openssl pkcs8 -topk8 -nocrypt -traditional`

# binary version of this package (-bin): github.com/noahvogt/chromium-extension-copy-url-on-hover-bin-aur

_extension=copy-url-on-hover
pkgname=chromium-extension-"$_extension"
pkgver=0.8.0
pkgrel=1
pkgdesc="Copy URL On Hover - chromium extension"
arch=('any')
url="https://github.com/noahvogt/$_extension"
license=('custom:none')
depends=('chromium')
makedepends=('openssl' 'jq')
source=("https://github.com/noahvogt/$_extension/archive/refs/tags/v$pkgver.tar.gz"
        "$_extension.pem")
sha256sums=('8193a3c887467ac8d3148835fe3afa14d053c51760e82832be09ee57e0b95ba8'
            'c06aca0d925d5f6f8ab55e7c3032fee7ffeb56c99fd3625e90bd167bef489dfc')

build() {
    # prepare source directory (while retaining reproducibility)
    mv "$_extension-$pkgver" "$pkgname-$pkgver"
    find "$pkgname-$pkgver" -exec touch -t 202403120000 {} +

    # derive variables from private key
    pubkey="$(openssl rsa -in "$_extension.pem" -pubout -outform DER | base64 -w0)"
    export _id="$(echo $pubkey | base64 -d | sha256sum | head -c32 | tr '0-9a-f' 'a-p')"

    # create extension json
    _extver="$(jq -r '.version' "$pkgname-$pkgver/manifest.json")"
    cat << EOF > "$_id".json
{
    "external_crx": "/usr/lib/$pkgname/$pkgname-$pkgver.crx",
    "external_version": "$_extver"
}
EOF

    # enroll public key in manifest
    cd "$pkgname-$pkgver"
    jq --ascii-output --arg key "$pubkey" '. + {key: $key}' manifest.json > manifest.json.new
    mv manifest.json.new manifest.json
    touch -t 202403120000 manifest.json
    cd ..

    # pack extension
    tmpdir="$(mktemp -d chromium-pack-XXXXXX)"
    chromium --user-data-dir="$tmpdir" --pack-extension="$pkgname-$pkgver" --pack-extension-key="$_extension".pem
}

package() {
    install -Dm644 -t "$pkgdir/usr/share/chromium/extensions/" "$_id.json"
    install -Dm644 -t "$pkgdir/usr/lib/$pkgname/" "$pkgname-$pkgver.crx"
}
