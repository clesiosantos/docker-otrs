FROM alpine:3.7
LABEL maintainer="Clesio Santos"

## Set Defaults
  ENV OTRS_VERSION=6.0.4 \
       ZABBIX_HOSTNAME=otrs \
       DB_HOST=db \
       DB_NAME=otrs \
       DB_USER=root \
       DB_PASS=otrs \
       ORGANIZATION="Docker OTRS" \
       ROOT_PASS=otrs \
       ADMIN_EMAIL=sysadmin@email.com \
       NGINX_VERSION=1.14.0

### Install packages
  RUN GPG_KEYS=B0F4253373F8F6F510D42178520A9993A1C052F8 \
	&& CONFIG="\
		--prefix=/etc/nginx \
		--sbin-path=/usr/sbin/nginx \
		--modules-path=/usr/lib/nginx/modules \
		--conf-path=/etc/nginx/nginx.conf \
		--error-log-path=/var/log/nginx/error.log \
		--http-log-path=/var/log/nginx/access.log \
		--pid-path=/var/run/nginx.pid \
		--lock-path=/var/run/nginx.lock \
		--http-client-body-temp-path=/var/cache/nginx/client_temp \
		--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
		--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
		--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
		--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
		--user=nginx \
		--group=nginx \
		--with-http_ssl_module \
		--with-http_realip_module \
		--with-http_addition_module \
		--with-http_sub_module \
		--with-http_dav_module \
		--with-http_flv_module \
		--with-http_mp4_module \
		--with-http_gunzip_module \
		--with-http_gzip_static_module \
		--with-http_random_index_module \
		--with-http_secure_link_module \
		--with-http_stub_status_module \
		--with-http_auth_request_module \
		--with-http_xslt_module=dynamic \
		--with-http_image_filter_module=dynamic \
		--with-http_geoip_module=dynamic \
		--with-http_perl_module=dynamic \
		--with-threads \
		--with-stream \
		--with-stream_ssl_module \
		--with-stream_ssl_preread_module \
		--with-stream_realip_module \
		--with-stream_geoip_module=dynamic \
		--with-http_slice_module \
		--with-mail \
		--with-mail_ssl_module \
		--with-compat \
		--with-file-aio \
		--with-http_v2_module \
	" \
	&& addgroup -S nginx \
	&& adduser -D -S -h /var/cache/nginx -s /sbin/nologin -G nginx nginx \
	&& apk add --no-cache --virtual .build-deps \
		gcc \
		libc-dev \
		make \
		openssl-dev \
		pcre-dev \
		zlib-dev \
		linux-headers \
		curl \
		gnupg \
		libxslt-dev \
		gd-dev \
		geoip-dev \
		perl-dev \
	&& curl -fSL https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz -o nginx.tar.gz \
	&& curl -fSL https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz.asc  -o nginx.tar.gz.asc \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& found=''; \
	for server in \
		ha.pool.sks-keyservers.net \
		hkp://keyserver.ubuntu.com:80 \
		hkp://p80.pool.sks-keyservers.net:80 \
		pgp.mit.edu \
	; do \
		echo "Fetching GPG key $GPG_KEYS from $server"; \
		gpg --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$GPG_KEYS" && found=yes && break; \
	done; \
	test -z "$found" && echo >&2 "error: failed to fetch GPG key $GPG_KEYS" && exit 1; \
	gpg --batch --verify nginx.tar.gz.asc nginx.tar.gz \
	&& rm -rf "$GNUPGHOME" nginx.tar.gz.asc \
	&& mkdir -p /usr/src \
	&& tar -zxC /usr/src -f nginx.tar.gz \
	&& rm nginx.tar.gz \
	&& cd /usr/src/nginx-$NGINX_VERSION \
	&& ./configure $CONFIG --with-debug \
	&& make -j$(getconf _NPROCESSORS_ONLN) \
	&& mv objs/nginx objs/nginx-debug \
	&& mv objs/ngx_http_xslt_filter_module.so objs/ngx_http_xslt_filter_module-debug.so \
	&& mv objs/ngx_http_image_filter_module.so objs/ngx_http_image_filter_module-debug.so \
	&& mv objs/ngx_http_geoip_module.so objs/ngx_http_geoip_module-debug.so \
	&& mv objs/ngx_http_perl_module.so objs/ngx_http_perl_module-debug.so \
	&& mv objs/ngx_stream_geoip_module.so objs/ngx_stream_geoip_module-debug.so \
	&& ./configure $CONFIG \
	&& make -j$(getconf _NPROCESSORS_ONLN) \
	&& make install \
	&& rm -rf /etc/nginx/html/ \
	&& mkdir /etc/nginx/conf.d/ \
	&& mkdir -p /usr/share/nginx/html/ \
	&& install -m644 html/index.html /usr/share/nginx/html/ \
	&& install -m644 html/50x.html /usr/share/nginx/html/ \
	&& install -m755 objs/nginx-debug /usr/sbin/nginx-debug \
	&& install -m755 objs/ngx_http_xslt_filter_module-debug.so /usr/lib/nginx/modules/ngx_http_xslt_filter_module-debug.so \
	&& install -m755 objs/ngx_http_image_filter_module-debug.so /usr/lib/nginx/modules/ngx_http_image_filter_module-debug.so \
	&& install -m755 objs/ngx_http_geoip_module-debug.so /usr/lib/nginx/modules/ngx_http_geoip_module-debug.so \
	&& install -m755 objs/ngx_http_perl_module-debug.so /usr/lib/nginx/modules/ngx_http_perl_module-debug.so \
	&& install -m755 objs/ngx_stream_geoip_module-debug.so /usr/lib/nginx/modules/ngx_stream_geoip_module-debug.so \
	&& ln -s ../../usr/lib/nginx/modules /etc/nginx/modules \
	&& strip /usr/sbin/nginx* \
	&& strip /usr/lib/nginx/modules/*.so \
	&& rm -rf /usr/src/nginx-$NGINX_VERSION \
	\
	# Bring in gettext so we can get `envsubst`, then throw
	# the rest away. To do this, we need to install `gettext`
	# then move `envsubst` out of the way so `gettext` can
	# be deleted completely, then move `envsubst` back.
	&& apk add --no-cache --virtual .gettext gettext \
	&& mv /usr/bin/envsubst /tmp/ \
	\
	&& runDeps="$( \
		scanelf --needed --nobanner /usr/sbin/nginx /usr/lib/nginx/modules/*.so /tmp/envsubst \
			| awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
			| sort -u \
			| xargs -r apk info --installed \
			| sort -u \
	)" \
	&& apk add --no-cache --virtual .nginx-rundeps $runDeps \
	&& apk del .build-deps \
	&& apk del .gettext \
	&& mv /tmp/envsubst /usr/local/bin/ \
	\
	# Bring in tzdata so users could set the timezones through the environment
	# variables
	&& apk add --no-cache tzdata \
	\
	# forward request and error logs to docker log collector
	&& ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log \
        && echo http://dl-4.alpinelinux.org/alpine/edge/testing >> /etc/apk/repositories && \
        apk update && \
        apk add \
            encodings \
            expat \
            fcgi \
            fcgiwrap \
            fontconfig \
            freetype \
            libbz2 \
            libfontenc \
            libgcrypt \
            libgpg-error \
            libidn \
            libpng \
            libxml2 \
            libxslt \
            mariadb-client \
            mkfontdir \
            mkfontscale \
            perl \
            perl-appconfig \
            perl-archive-zip \
            perl-canary-stability \
            perl-capture-tiny \
            perl-cgi \
            perl-cgi-fast \
            perl-class-mix \
            perl-common-sense \
            perl-convert-asn1 \
            perl-crypt-eksblowfish \
            perl-crypt-ssleay \
            perl-datetime \
            perl-date-format \
            perl-dbd-mysql \
            perl-dbi \
            perl-devel-symdump \
            perl-encode-hanextra \
            perl-encode-locale \
            perl-fcgi \
            perl-file-listing \
            perl-html-parser \
            perl-html-tagset \
            perl-http-cookies \
            perl-http-daemon \
            perl-http-date \
            perl-http-message \
            perl-http-negotiate \
            perl-io-html \
            perl-io-socket-ssl \
            perl-json-xs \
            perl-ldap \
            perl-libwww \
            perl-lwp-mediatypes \
            perl-mail-imapclient \
            perl-net-dns \
            perl-net-http \
            perl-net-libidn \
            perl-net-ssleay \
            perl-params-classify \
            perl-parse-recdescent \
            perl-path-class \
            perl-pod-coverage \
            perl-template-toolkit \
            perl-test-pod \
            perl-text-csv \
            perl-text-csv_xs \
            perl-try-tiny \
            perl-types-serialiser \
            perl-uri \
            perl-www-robotrules \
            perl-xml-libxml \
            perl-xml-libxslt \
            perl-xml-namespacesupport \
            perl-xml-parser \
            perl-xml-sax \
            perl-xml-sax-base \
            perl-yaml-libyaml \
            spawn-fcgi \
            ttf-dejavu \
            && \


### Install OTRS
        mkdir -p /opt/otrs && \
        echo "Downloading & Extracting OTRS archive otrs-${OTRS_VERSION}.tar.gz" && \
        curl http://ftp.otrs.org/pub/otrs/otrs-${OTRS_VERSION}.tar.gz |tar -xzf - -C /opt/otrs --strip 1 && \
       
### Add User
       adduser -D -S -h /opt/otrs -s /sbin/nologin -G www-data otrs && \

### Setup OTRS Directories
       mkdir -p /opt/otrs/var/run/ /opt/otrs/var/tmp && \
       mkdir -p /run/spawn-fcgi && \
    
### Set Permissions
       /opt/otrs/bin/otrs.SetPermissions.pl --otrs-user=otrs --web-group=www-data && \

### Setup Persistence
       mkdir -p /assets/otrs-config && \
       mv /opt/otrs/Kernel/ /assets/otrs-config/ && \
       ### Cleanup
        rm -rf /var/cache/apk/*

## validar
  COPY nginx.conf /etc/nginx/nginx.conf
  COPY nginx.vh.default.conf /etc/nginx/conf.d/default.conf



### Networking Configuration
  EXPOSE 80
  STOPSIGNAL SIGTERM

### Files Add
  ADD install /
   
  CMD ["nginx", "-g", "daemon off;"]
