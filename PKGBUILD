# $Id: PKGBUILD 55082 2011-09-02 09:21:05Z spupykin $
# Maintainer: Sergej Pupykin <pupykin.s+arch@gmail.com>
# Contributor: Miroslaw Szot <mss@czlug.icis.pcz.pl>
# Contributor: Ludovic Fauvet <etix@l0cal.com>

_doc_root=/usr/share/nginx/http
_server_root=/etc/nginx
_conf_path=${_server_root}/conf
_tmp_path=/var/spool/nginx
_log_path=/var/log/nginx
_user=http
_group=http
_cachepurge_ver=1.3

pkgname=nginx-cache-purge
pkgver=1.0.6
pkgrel=3
pkgdesc="lightweight HTTP server and IMAP/POP3 proxy server with cache_purge support"
arch=('i686' 'x86_64')
depends=('pcre' 'zlib' 'openssl')
url="http://nginx.org"
conflicts=('nginx')
provides=('nginx')
license=('custom')
backup=("etc/nginx/conf/fastcgi.conf"
	"etc/nginx/conf/fastcgi_params"
	"etc/nginx/conf/koi-win"
	"etc/nginx/conf/koi-utf"
	"etc/nginx/conf/mime.types"
	"etc/nginx/conf/nginx.conf"
	"etc/nginx/conf/scgi_params"
	"etc/nginx/conf/uwsgi_params"
	"etc/nginx/conf/win-utf"
	"etc/logrotate.d/nginx"
	"etc/conf.d/nginx")
changelog=changelog
source=(http://nginx.org/download/nginx-$pkgver.tar.gz
	    nginx
        http://labs.frickle.com/files/ngx_cache_purge-${_cachepurge_ver}.tar.gz)
md5sums=('bc98bac3f0b85da1045bc02e6d8fc80d'
         '0e8032d3ba26c3276e8c7c30588d375f'
         '7e307f58ce4bf7a2bb2899a162ae10ca')

build() {
	local _src_dir=$srcdir/nginx-${pkgver}
	local _build_dir=$_src_dir/objs
    local _cachepurge_dirname=ngx_cache_purge-${_cachepurge_ver}

	cd $_src_dir
	./configure \
		--prefix=${_server_root} \
		--sbin-path=/usr/sbin/nginx \
		--pid-path=/var/run/nginx.pid \
		--lock-path=/var/lock/nginx.lock \
		--http-client-body-temp-path=${_tmp_path}/client_body_temp \
		--http-proxy-temp-path=${_tmp_path}/proxy_temp \
		--http-fastcgi-temp-path=${_tmp_path}/fastcgi_temp \
		--http-log-path=${_log_path}/access.log \
		--error-log-path=${_log_path}/error.log \
		--user=${_user} --group=${_group} \
		--with-imap --with-imap_ssl_module --with-http_ssl_module \
		--with-http_stub_status_module \
		--with-http_dav_module \
		--with-http_gzip_static_module \
		--with-ipv6 \
        --add-module=../${_cachepurge_dirname}

	make
}

package() {
	cd $srcdir/nginx-${pkgver}
	make DESTDIR=$pkgdir install

	install -d $pkgdir/etc/logrotate.d/
	cat <<EOF > $pkgdir/etc/logrotate.d/nginx
	$_log_path/*log {
		create 640 http log
		compress
		postrotate
			/bin/kill -USR1 \`cat /var/run/nginx.pid 2>/dev/null\` 2> /dev/null || true
		endscript
	}
EOF

	sed -i -e "s/\<user\s\+\w\+;/user $_user;/g" $pkgdir/$_conf_path/nginx.conf

	install -d $pkgdir/$_tmp_path

	# move default document root outside server root
	install -d $pkgdir/$_doc_root
	mv $pkgdir/$_server_root/html/* $pkgdir/$_doc_root/
	rm -rf $pkgdir/$_server_root/html
	rm -f $pkgdir/$_doc_root/index.html

	# let's create links for relative paths in config file
	ln -s $_log_path $pkgdir/$_server_root/logs
	ln -s $_doc_root $pkgdir/$_server_root/html

	install -D -m755 $srcdir/nginx $pkgdir/etc/rc.d/nginx
	install -D -m644 LICENSE $pkgdir/usr/share/licenses/nginx/LICENSE
	mkdir -p $pkgdir/etc/conf.d
	echo "NGINX_CONFIG=/etc/nginx/conf/nginx.conf" >$pkgdir/etc/conf.d/nginx
	rm -rf $pkgdir/var/run
}
