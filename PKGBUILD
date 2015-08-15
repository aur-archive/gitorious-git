# Maintainer: Massimiliano Torromeo <massimiliano.torromeo@gmail.com>

pkgname=gitorious-git
pkgver=20130313
pkgrel=1
pkgdesc="Gitorious aims to provide a great way of doing distributed opensource code collaboration (git version)"
arch=(i686 x86_64)
url="http://gitorious.org/gitorious"
license=('AGPLv3')

depends=('mysql' 'ruby-bundler' 'imagemagick' 'nodejs-buster' 'libxslt')
makedepends=('git')
optdepends=(
	'memcached: High performance cache'
	'redis: message queue'
	'sphinx: search')
options=('!strip')

install=gitorious.install
backup=(
	etc/webapps/gitorious/gitorious.yml
	etc/webapps/gitorious/database.yml
	etc/webapps/gitorious/authentication.yml
	etc/webapps/gitorious/memcache.yml
	etc/webapps/gitorious/resque.yml
	etc/webapps/gitorious/unicorn.rb
)

source=(
	gitorious.service
	gitorious-resque.service
	gitorious-sphinx.service
)

conflicts=(gitorious)
provides=(gitorious)

_gitroot="git://gitorious.org/gitorious/mainline.git"
_gitname="gitorious"

build() {
	cd "$srcdir"

	# trick systemtimer build system into thinking we have gcc-4.6
	rm -rf gcc46-pathfix
	mkdir gcc46-pathfix
	ln -s /usr/bin/gcc gcc46-pathfix/gcc-4.6
	export PATH="$PATH:$srcdir/gcc46-pathfix"

	msg2 "Connecting to GIT server...."

	if [ -d "$srcdir/$_gitname" ]; then
		cd $_gitname && git pull origin
		msg "The local files are updated."
	else
		git clone -b next $_gitroot $_gitname
		cd $_gitname
	fi
	
	# culljs git url update
	cd public/ui3
	git config submodule.lib/culljs.url https://github.com/culljs/culljs.git
	cd ../..
	
	git submodule update --recursive --init

	msg2 "GIT checkout done or server timeout"

	rm -rf "$srcdir/$_gitname-build"
	cp -r "$srcdir/$_gitname" "$srcdir/$_gitname-build"
	cd "$srcdir/$_gitname-build/"
	rm -rf .git
	find . -type f -name .gitignore -delete

	msg2 "Fetching bundled gems..."
	bundle install --deployment --binstubs
	rm -rf vendor/bundle/ruby/*/cache
}

package() {
	cd "$srcdir"

	_gitorious="/usr/share/webapps/gitorious"
	_etc="$pkgdir/etc/webapps/gitorious"
	install -d "$pkgdir/usr/share/webapps"
	install -d "$_etc"

	msg2 "Copying gitorious files"
	cp -a "$_gitname-build" "${pkgdir}${_gitorious}"

	# Configurations
	msg2 "Moving configuration files"
	for f in "$pkgdir$_gitorious/config/"*.sample*; do
		fdest=$(basename "$f")
		fdest=${fdest/.sample/}
		install -Dm0644 "$f" "$_etc/$fdest"
		rm "$f"
		ln -s "/etc/webapps/gitorious/$fdest" "$pkgdir$_gitorious/config/$fdest"
	done

	sed -e 's|^  password:\s*$|  password: G170r!oUs|' \
	    -e 's|^  username:.*$|  username: gitorious|' \
	    -e 's|gitorious_production|gitorious|' \
	    -i "$_etc/database.yml"

	sed -e 's|/var/git/|/var/gitorious/|' \
	    -e 's|gitorious.local|localhost.localdomain|' \
	    -e 's|/var/gitorious/repositories|/srv/git|' \
	    -i "$_etc/gitorious.yml"

	echo -e "production:\n  - localhost:11211\n" > "$_etc/memcache.yml"

	# capillary
	ln -s /usr/lib/node_modules/buster/node_modules/ "${pkgdir}${_gitorious}/public/javascripts/lib/capillary/node_modules"

	# services
	msg2 "Installing services"
	install -Dm0755 "$srcdir/gitorious.service" "$pkgdir/usr/lib/systemd/system/gitorious.service"
	install -Dm0755 "$srcdir/gitorious-resque.service" "$pkgdir/usr/lib/systemd/system/gitorious-resque.service"
	install -Dm0755 "$srcdir/gitorious-sphinx.service" "$pkgdir/usr/lib/systemd/system/gitorious-sphinx.service"

	# data directories
	msg2 "Setting up data directories"
	install -dm0755 "$pkgdir/var/gitorious/"{tarballs,tarballs-work}

	# remove tests
	msg2 "Removing tests"
	find "$pkgdir" -type d \( -name test -o -name tests \) -exec rm -rf {} +
}

sha256sums=('f13b9e36765ce20819eb1cb15bb1d392629fd5956d0757e9f3bc5a3503afe4f3'
            '224830b4c0201d8cc233bf8e205d1d86e82cf23a0aafcd02c242e5a63a7b3199'
            'e9a694187b872b2f03507a347d70007023d6d091ccc2a77a9d98bb95f4ddb4e4')
