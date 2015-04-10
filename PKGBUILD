# Contributor: Jonas Heinrich <onny@project-insanity.org>
# Maintainer: Jonas Heinrich <onny@project-insanity.org>

pkgname=sharelatex-ldap
pkgver=master
pkgrel=1
pkgdesc="A web-based collaborative LaTeX editor"
arch=('any')
url="www.sharelatex.com"
license=('GPL3')
depends=('nodejs=0.10.37' 'redis' 'mongodb' 'texlive-core' 'aspell' 'nodejs-grunt')

conflicts=('sharelatex')
replaces=('sharelatex')
provides=('sharelatex')
_server=''
if [[ -n $(which httpd 2> /dev/null) ]]; then
	backup=('etc/httpd/conf/extra/gitlab.conf'
		'etc/httpd/conf/extra/gitlab-ssl.conf')
	_server='apache'
fi
if [[ -n $(which nginx 2> /dev/null) ]]; then
	backup=('etc/nginx/sites-available/gitlab-ssl')
	_server='nginx'
fi
if [[ -n $(which lighttpd 2> /dev/null) ]]; then
	backup=('etc/lighttpd/conf.d/10-gitlab.conf')
	_server='lighttpd'
fi
backup=('etc/webapps/sharelatex/settings.development.coffee')
source=("git+https://github.com/sharelatex/sharelatex.git")
#install='gitlab.install'
sha512sums=("SKIP")
options=('!strip')

_ldaprepo="https://github.com/heukirne/web-sharelatex.git"
_ldapbranch='ldap-auth'
_homedir="/var/lib/sharelatex"
_datadir="/usr/share/webapps/sharelatex"
_wo=""

pkgver() {
	cd "$SRCDEST/${_pkgname}"
	git describe --always | sed 's|-|.|g'
}

prepare() {
	cd "${srcdir}/sharelatex"

	# Patching config files:
	sed -i "/name:[[:space:]]\"web\"/{N;N;s|\(repo:[[:space:]]*\"\).*\(\"\n.*\)|\1${_ldaprepo}\2|;s|\(version:[[:space:]]\"\).*\(\".*\)|\1${_ldapbranch}\2|}" Gruntfile.coffee
}
build() {
	cd "${srcdir}/sharelatex"
	msg "npm install..."
	npm install
	msg "grunt install..."
	grunt install
}

package() {
	cd "${srcdir}/sharelatex"
	install -d "${pkgdir}/usr/share/webapps"
	cp -r "${srcdir}/sharelatex" "${pkgdir}${_datadir}"

	# Creating directories
	install -d \
		"${pkgdir}/etc/webapps/sharelatex" \
		"${pkgdir}/usr/share/webapps" \
		"${pkgdir}/usr/share/doc/${pkgname}" \
		"${pkgdir}${_datadir}/public/uploads"

	# Install config files
	for _service in chat clsi docstore document-updater filestore spelling tags track-changes web; do
		mv "${_service}/config/settings.defaults.coffee" "${pkgdir}/etc/webapps/sharelatex/${_service}.conf.coffee"
		[[ -f "${pkgdir}${_datadir}/${_service}/config/settings.defaults.coffee" ]] && rm "${pkgdir}${_datadir}/${_service}/config/settings.defaults.coffee"
		ln -s "/etc/webapps/sharelatex/${_service}.conf.coffee" "${pkgdir}${_datadir}/${_service}/config/settings.defaults.coffee"
    done

	# Install main config file
	mv "config/settings.development.coffee" "${pkgdir}/etc/webapps/sharelatex/sharelatex.conf.coffee"
	[[ -f "${pkgdir}${_datadir}/config/settings.development.coffee" ]] && rm "${pkgdir}${_datadir}/config/settings.development.coffee"
	ln -s "/etc/webapps/sharelatex/sharelatex.conf.coffee" "${pkgdir}${_datadir}/config/settings.development.coffee"

	# Install license and help files
	mv README.md CONTRIBUTING.md CHANGELOG.md config/*.{example} "${pkgdir}/usr/share/doc/${pkgname}"
	install -D "LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
	rm "${pkgdir}/usr/share/webapps/sharelatex/LICENSE"

	install -Dm0755 "${srcdir}/sharelatex@.service" "${pkgdir}/usr/lib/systemd/system/sharelatex@.service"
	install -Dm0755 "${srcdir}/start_module.sh" "${pkgdir}/etc/sharelatex/start_module.sh"

	case ${_server} in
		apache)
			install -d "${pkgdir}/etc/httpd/conf/extra"
			install -m 644 "${srcdir}/sharelatex.conf" "${pkgdir}/etc/httpd/conf/extra/"
			;;
		nginx)
			install -d "${pkgdir}/etc/nginx/sites-enabled"
			install -m 644 "${srcdir}/gitlab_nginx" "${pkgdir}/etc/nginx/sites-enabled/"
			;;
	esac
}

# vim:set ts=4 sw=4 et:
