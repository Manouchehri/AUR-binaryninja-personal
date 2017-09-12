# Maintainer: David Manouchehri
# Contributor: Alex Palaistras <alex+archlinux@deuill.org>
# Contributor: Elen Eisendle
# Contributor: Spenser Reinhardt

_pkgname="binaryninja"
_branch="stable"
_edition="demo" # demo, personal, professional (corporate not tested)
pkgname="${_pkgname}-${_edition}"
[[ "${_branch}" != "stable" ]] && pkgname="${pkgname}-${_branch}"
pkgdesc="Binary Ninja is a binary multi-tool and reversing platform"
url="https://binary.ninja"
license=('custom:Binary Ninja License Agreement')
arch=('x86_64')
conflicts=("${_pkgname}")
provides=("${_pkgname}")
pkgver=1.1.922
pkgrel=2 # reset after new release, and .srcinfo
install="${_pkgname}.install"
makedeps=('curl' 'perl')
depends=(
	'python2' 'glibc' 'glib2' 'gcc-libs-multilib' 'pcre' 'zlib'
	'libssh2' 'libnghttp2' 'libpsl' 'libxcb' 'icu' 'keyutils'
	'libxext' 'libx11' 'libglvnd' 'krb5' 'e2fsprogs' 'libffi'
	'libxau' 'libxdmcp' 'libcurl-compat' 'openssl-1.0'
	'qt5-base'
)
# https://binary.ninja/recover/
if [[ "${_edition}" == "professional" ]]; then
	_hash="68b6183251c20cfbcc37e99007aa09b565988c1b8d7a2c46e7c9fcd7c6f5f850"
	_src="file://BinaryNinja_linux.zip"
elif [[ "${_edition}" == "demo" ]]; then
	_src="https://cdn.binary.ninja/installers/BinaryNinja-${_edition}.zip"
else
	_src="file://BinaryNinja-${_edition}.zip"
fi
[ -z ${_hash} ] && _hash=$(curl -s https://binary.ninja/js/hashes.js | perl -pe "s/.*Ninja-${_edition}.zip\":\s\"([\da-f]+)\".*/\$1/g")

source=(
	"${_src}"
	"${_pkgname}.png"
)

sha256sums=(
	"${_hash}"
	'ac2e652f617d5ef8aaa34a5113164f51f3f673c872a635d29c93878a00650bf8'
)

pkgver() {
	curl -s https://binary.ninja/js/changelog.js | perl -pe 's/.*?version":\s"(\d+\.\d+\.\d+)".*/$1/'
}

_cp_files() {
	mode="${1}"
	inpath="${2}" # single path
	outpath="${3}"
	[[ ${#} -gt 3 ]] && depth="${4}" # limit
	[[ ${#} -gt 4 ]] && skip="${5}" # comma separted list
	# set $cmd based on depth's existance
	[ -z ${depth+x} ] && cmd="find ${inpath} -type f" || cmd="find ${inpath} -maxdepth ${depth} -type f"
	for file in $(${cmd}); do
		if [ ! -z ${skip} ]; then # dont bother if unset
			for s in $(echo ${skip}| sed 's/,/ /g'); do
				[[ "${file}" =~ "${s}" ]] && c=1 && break
			done
			[ ! -z ${c+x} ] && unset c && continue # is skipped ? unset+continue : move file
		fi
		outfile=$(echo ${file} | perl -pe "s|${srcdir}/${_pkgname}|${outpath}|g") # trim
		install -m "${mode}" "${file}" "${outfile}"
	done
}

prepare() {
	echo "[Desktop Entry]
Name=Binary Ninja ${_edition}
Exec=/usr/bin/${_pkgname}
Icon=${_pkgname}
Type=Application
Categories=Development;Debugger;Profiling;" > "${srcdir}/${_pkgname}.desktop"
}

package() {
	_srcdir="${srcdir}/${_pkgname}"
	destdir="${pkgdir}/opt/${_pkgname}"
	msg2 "${_srcdir}"
	msg2 "${destdir}"
	[[ "${_branch}" != "stable" ]] && destdir="${destdir}-${_branch}"
	
	msg2 "Creating directories"
	install -dm 755 "${pkgdir}/usr/share/icons/hicolor/128x128/apps/"
	install -dm 755 "${pkgdir}/usr/share/applications/"
	for dir in $(find "${_srcdir}" -type d); do
		dir=$(echo $dir | perl -pe "s|${_srcdir}||g")
		install -dm 755 "${destdir}/${dir}"
	done
	
	msg2 "Copying non-executable files"
	install -m 644 "${srcdir}/${_pkgname}.png" "${pkgdir}/usr/share/icons/hicolor/128x128/apps/"
	install -m 644 "${srcdir}/${_pkgname}.desktop" "${pkgdir}/usr/share/applications/"
	install -m 644 "${_srcdir}/qt.conf" "${destdir}/qt.conf"
	_cp_files 644 "${_srcdir}/types" "${destdir}"
	if [[ "${_edition}" != "demo" ]]; then
		_cp_files 644 "${_srcdir}/docs" "${destdir}"
		_cp_files 644 "${_srcdir}/api-docs" "${destdir}"
		_cp_files 644 "${_srcdir}/scc-docs" "${destdir}"
	fi
	
	msg2 "Copying executable files"
	_cp_files 755 "${_srcdir}/*" "${destdir}" 0 'qt.conf,libQt5'
	_cp_files 755 "${_srcdir}/plugins" "${destdir}"
	if [[ "${_edition}" != "demo" ]]; then
		_cp_files 755 "${_srcdir}/python" "${destdir}"
		_cp_files 755 "${_srcdir}/scripts" "${destdir}"
		_cp_files 755 "${_srcdir}/examples" "${destdir}"
	fi
}
# vim:set et sw=2 sts=2 tw=80:
