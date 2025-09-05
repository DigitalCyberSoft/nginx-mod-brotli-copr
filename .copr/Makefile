# COPR Makefile for automated nginx-mod-brotli builds
# Downloads ngx_brotli from GitHub and builds as nginx dynamic module

# COPR provides outdir, set default for local testing
outdir ?= .

# Default nginx version if detection fails
NGINX_VERSION ?= 1.28.0

# ngx_brotli version (using latest release)
BROTLI_VERSION = v1.0.0rc

# Default target for COPR
srpm:
	@echo "=== Starting nginx-mod-brotli SRPM build ==="
	@echo "Output directory: $(outdir)"
	@echo "Working directory: $$(pwd)"
	
	# Install git if not available (for COPR environment)
	@if ! command -v git >/dev/null 2>&1; then \
		echo "=== Installing git ==="; \
		if command -v dnf >/dev/null 2>&1; then \
			dnf install -y git || exit 1; \
		elif command -v yum >/dev/null 2>&1; then \
			yum install -y git || exit 1; \
		else \
			echo "ERROR: No package manager found to install git"; \
			exit 1; \
		fi; \
	fi
	
	# Try to detect nginx version from repos
	@if command -v dnf >/dev/null 2>&1; then \
		echo "=== Detecting nginx version from repos ==="; \
		NGINX_VER=$$(dnf repoquery --latest-limit=1 --qf="%{version}" nginx 2>/dev/null || echo "$(NGINX_VERSION)"); \
	else \
		echo "=== Using default nginx version ==="; \
		NGINX_VER="$(NGINX_VERSION)"; \
	fi; \
	echo "Using nginx version: $$NGINX_VER"; \
	\
	echo "=== Downloading nginx source ==="; \
	if ! curl -L -o nginx-$$NGINX_VER.tar.gz http://nginx.org/download/nginx-$$NGINX_VER.tar.gz 2>/dev/null; then \
		echo "Failed to download nginx $$NGINX_VER, trying $(NGINX_VERSION)..."; \
		curl -L -o nginx-$(NGINX_VERSION).tar.gz http://nginx.org/download/nginx-$(NGINX_VERSION).tar.gz || exit 1; \
		NGINX_VER="$(NGINX_VERSION)"; \
	fi; \
	\
	echo "=== Downloading ngx_brotli $(BROTLI_VERSION) ==="; \
	curl -L -o ngx_brotli-$(BROTLI_VERSION).tar.gz https://github.com/google/ngx_brotli/archive/$(BROTLI_VERSION).tar.gz || exit 1; \
	\
	echo "=== Extracting ngx_brotli to get submodules info ==="; \
	tar xzf ngx_brotli-$(BROTLI_VERSION).tar.gz; \
	cd ngx_brotli-*/ && git init && git remote add origin https://github.com/google/ngx_brotli.git && \
	git fetch --depth=1 origin $(BROTLI_VERSION) && git checkout FETCH_HEAD && \
	git submodule update --init --recursive && cd ..; \
	tar czf ngx_brotli-$(BROTLI_VERSION)-full.tar.gz ngx_brotli-*/ || exit 1; \
	rm -rf ngx_brotli-*/; \
	\
	echo "=== Generating spec file ==="; \
	( \
	echo '%global _hardened_build 1'; \
	echo '%global nginx_user nginx'; \
	echo ''; \
	echo '%ifnarch s390 s390x'; \
	echo '%global with_gperftools 1'; \
	echo '%endif'; \
	echo ''; \
	echo '%global with_aio 1'; \
	echo ''; \
	echo "%define nginx_ver $$NGINX_VER"; \
	echo '%define brotli_ver $(BROTLI_VERSION)'; \
	echo ''; \
	echo 'Name:              nginx-mod-brotli'; \
	echo 'Epoch:             1'; \
	echo 'Version:           %{nginx_ver}'; \
	echo 'Release:           1%{?dist}'; \
	echo ''; \
	echo 'Summary:           Brotli compression module for nginx'; \
	echo 'License:           BSD-2-Clause'; \
	echo 'URL:               https://github.com/google/ngx_brotli'; \
	echo ''; \
	echo 'Source0:           nginx-%{nginx_ver}.tar.gz'; \
	echo 'Source1:           ngx_brotli-%{brotli_ver}-full.tar.gz'; \
	echo ''; \
	echo 'BuildRequires:     gcc-c++'; \
	echo 'BuildRequires:     make'; \
	echo 'BuildRequires:     openssl-devel'; \
	echo 'BuildRequires:     pcre-devel'; \
	echo 'BuildRequires:     pcre2-devel'; \
	echo 'BuildRequires:     zlib-devel'; \
	echo 'BuildRequires:     cmake'; \
	echo ''; \
	echo '%if 0%{?with_gperftools}'; \
	echo 'BuildRequires:     gperftools-devel'; \
	echo '%endif'; \
	echo ''; \
	echo 'BuildRequires:     gd-devel'; \
	echo 'BuildRequires:     perl-devel'; \
	echo 'BuildRequires:     perl-generators'; \
	echo 'BuildRequires:     perl(ExtUtils::Embed)'; \
	echo 'BuildRequires:     libxslt-devel'; \
	echo 'BuildRequires:     systemd'; \
	echo ''; \
	echo 'Requires:          nginx = %{epoch}:%{nginx_ver}'; \
	echo 'Conflicts:         nginx > %{epoch}:%{nginx_ver}'; \
	echo 'Conflicts:         nginx < %{epoch}:%{nginx_ver}'; \
	echo 'Requires(post):    systemd'; \
	echo ''; \
	echo '%description'; \
	echo 'Brotli is a generic-purpose lossless compression algorithm that compresses'; \
	echo 'data using a combination of a modern variant of the LZ77 algorithm, Huffman'; \
	echo 'coding and 2nd order context modeling. This nginx module provides Brotli'; \
	echo 'compression support for nginx.'; \
	echo ''; \
	echo '%prep'; \
	echo '%setup -n nginx-%{nginx_ver} -q -a 1'; \
	echo 'mv ngx_brotli-* ngx_brotli'; \
	echo ''; \
	echo '%build'; \
	echo 'export DESTDIR=%{buildroot}'; \
	echo './configure \'; \
	echo '    --prefix=%{_datadir}/nginx \'; \
	echo '    --sbin-path=%{_sbindir}/nginx \'; \
	echo '    --modules-path=%{_libdir}/nginx/modules \'; \
	echo '    --conf-path=%{_sysconfdir}/nginx/nginx.conf \'; \
	echo '    --error-log-path=%{_localstatedir}/log/nginx/error.log \'; \
	echo '    --http-log-path=%{_localstatedir}/log/nginx/access.log \'; \
	echo '    --http-client-body-temp-path=%{_localstatedir}/lib/nginx/tmp/client_body \'; \
	echo '    --http-proxy-temp-path=%{_localstatedir}/lib/nginx/tmp/proxy \'; \
	echo '    --http-fastcgi-temp-path=%{_localstatedir}/lib/nginx/tmp/fastcgi \'; \
	echo '    --http-uwsgi-temp-path=%{_localstatedir}/lib/nginx/tmp/uwsgi \'; \
	echo '    --http-scgi-temp-path=%{_localstatedir}/lib/nginx/tmp/scgi \'; \
	echo '    --pid-path=/run/nginx.pid \'; \
	echo '    --lock-path=/run/lock/subsys/nginx \'; \
	echo '    --user=%{nginx_user} \'; \
	echo '    --group=%{nginx_user} \'; \
	echo '    --with-compat \'; \
	echo '    --with-debug \'; \
	echo '%if 0%{?with_aio}'; \
	echo '    --with-file-aio \'; \
	echo '%endif'; \
	echo '%if 0%{?with_gperftools}'; \
	echo '    --with-google_perftools_module \'; \
	echo '%endif'; \
	echo '    --with-http_ssl_module \'; \
	echo '    --with-http_v2_module \'; \
	echo '    --with-http_realip_module \'; \
	echo '    --with-http_addition_module \'; \
	echo '    --with-http_xslt_module=dynamic \'; \
	echo '    --with-http_image_filter_module=dynamic \'; \
	echo '    --with-http_sub_module \'; \
	echo '    --with-http_dav_module \'; \
	echo '    --with-http_flv_module \'; \
	echo '    --with-http_mp4_module \'; \
	echo '    --with-http_gunzip_module \'; \
	echo '    --with-http_gzip_static_module \'; \
	echo '    --with-http_auth_request_module \'; \
	echo '    --with-http_random_index_module \'; \
	echo '    --with-http_secure_link_module \'; \
	echo '    --with-http_degradation_module \'; \
	echo '    --with-http_slice_module \'; \
	echo '    --with-http_stub_status_module \'; \
	echo '    --with-http_perl_module=dynamic \'; \
	echo '    --with-mail=dynamic \'; \
	echo '    --with-mail_ssl_module \'; \
	echo '    --with-stream=dynamic \'; \
	echo '    --with-stream_ssl_module \'; \
	echo '    --with-stream_ssl_preread_module \'; \
	echo '    --with-pcre \'; \
	echo '    --with-pcre-jit \'; \
	echo '    --with-threads \'; \
	echo '    --add-dynamic-module=./ngx_brotli \'; \
	echo '    --with-cc-opt="%{optflags}" \'; \
	echo '    --with-ld-opt="$$RPM_LD_FLAGS -Wl,-E"'; \
	echo ''; \
	echo 'make %{?_smp_mflags} modules'; \
	echo ''; \
	echo '%install'; \
	echo 'install -d -m 0755 %{buildroot}%{_libdir}/nginx/modules'; \
	echo 'install -d -m 0755 %{buildroot}%{_datadir}/nginx/modules'; \
	echo ''; \
	echo 'install -m 0755 objs/ngx_http_brotli_filter_module.so %{buildroot}%{_libdir}/nginx/modules/'; \
	echo 'install -m 0755 objs/ngx_http_brotli_static_module.so %{buildroot}%{_libdir}/nginx/modules/'; \
	echo ''; \
	echo 'echo '\''load_module "%{_libdir}/nginx/modules/ngx_http_brotli_filter_module.so";'\'' \'; \
	echo '     > %{buildroot}%{_datadir}/nginx/modules/mod-brotli.conf'; \
	echo 'echo '\''load_module "%{_libdir}/nginx/modules/ngx_http_brotli_static_module.so";'\'' \'; \
	echo '     >> %{buildroot}%{_datadir}/nginx/modules/mod-brotli.conf'; \
	echo ''; \
	echo '%post'; \
	echo 'if [ $$1 -eq 1 ]; then'; \
	echo '    /usr/bin/systemctl reload nginx.service >/dev/null 2>&1 || :'; \
	echo 'fi'; \
	echo ''; \
	echo '%files'; \
	echo '%{_datadir}/nginx/modules/mod-brotli.conf'; \
	echo '%{_libdir}/nginx/modules/ngx_http_brotli_filter_module.so'; \
	echo '%{_libdir}/nginx/modules/ngx_http_brotli_static_module.so'; \
	echo ''; \
	echo '%changelog'; \
	echo "* $$(date '+%a %b %d %Y') COPR Builder - %{nginx_ver}-1"; \
	echo '- Automated build from GitHub'; \
	echo '- Built against nginx version %{nginx_ver}'; \
	echo '- Using ngx_brotli release %{brotli_ver}'; \
	) > nginx-mod-brotli.spec || exit 1; \
	echo "Spec file generated"; \
	\
	echo "=== Building SRPM ==="; \
	rpmbuild -bs \
		--define "_sourcedir $$(pwd)" \
		--define "_specdir $$(pwd)" \
		--define "_builddir $$(pwd)" \
		--define "_srcrpmdir $(outdir)" \
		--define "_rpmdir $$(pwd)" \
		nginx-mod-brotli.spec || exit 1; \
	\
	echo "=== SRPM build complete ==="; \
	ls -la $(outdir)/*.src.rpm

clean:
	@rm -f *.tar.gz *.spec *.src.rpm
	@rm -rf ngx_brotli-*/

.PHONY: srpm clean