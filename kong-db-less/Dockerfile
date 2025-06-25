# === STAGE 1: Builder ===
FROM registry.access.redhat.com/ubi8/ubi AS builder

ARG OPENRESTY_VERSION=1.25.3.1
ARG KONG_VERSION=3.6.1
ENV KONG_VERSION=${KONG_VERSION}

RUN dnf install -y \
    make gcc git unzip tar curl which wget hostname \
    readline-devel ncurses-devel libyaml-devel libffi-devel openssl-devel \
    perl lua sudo findutils pcre-devel && \
    dnf clean all

WORKDIR /tmp

# Build OpenResty
RUN curl -LO https://github.com/Kong/openresty-release-mirror/releases/download/${OPENRESTY_VERSION}/openresty-${OPENRESTY_VERSION}.tar.gz && \
    tar -zxvf openresty-${OPENRESTY_VERSION}.tar.gz && \
    cd openresty-${OPENRESTY_VERSION} && \
    ./configure --prefix=/usr/local/openresty \
        --with-luajit \
        --with-http_ssl_module \
        --with-http_realip_module && \
    make -j$(nproc) && make install

# Install LuaRocks
ENV LUAROCKS_VERSION=3.11.0
RUN wget https://luarocks.org/releases/luarocks-${LUAROCKS_VERSION}.tar.gz && \
    tar zxpf luarocks-${LUAROCKS_VERSION}.tar.gz && \
    cd luarocks-${LUAROCKS_VERSION} && \
    ./configure \
      --with-lua=/usr/local/openresty/luajit \
      --with-lua-include=/usr/local/openresty/luajit/include/luajit-2.1 && \
    make && make install

ENV LUA_DIR=/usr/local/openresty/luajit
ENV LUA_PATH="/root/.luarocks/share/lua/5.1/?.lua;/root/.luarocks/share/lua/5.1/?/init.lua;;"
ENV LUA_CPATH="/root/.luarocks/lib/lua/5.1/?.so;;"

# Download Kong source
RUN curl -LO https://github.com/Kong/kong/archive/refs/tags/${KONG_VERSION}.tar.gz && \
    tar -xzf ${KONG_VERSION}.tar.gz && \
    mv kong-${KONG_VERSION} /usr/local/kong

# === Required Kong Lua modules ===

# lua-resty-jit-uuid
RUN git clone https://github.com/thibaultcha/lua-resty-jit-uuid.git && \
    cd lua-resty-jit-uuid && \
    luarocks make --tree=/root/.luarocks --lua-dir=$LUA_DIR && \
    cd .. && rm -rf lua-resty-jit-uuid

# lua-resty-timer-ng
RUN git clone https://github.com/Kong/lua-resty-timer-ng.git && \
    cd lua-resty-timer-ng && \
    luarocks make --tree=/root/.luarocks --lua-dir=$LUA_DIR && \
    cd .. && rm -rf lua-resty-timer-ng

# lua-resty-ipmatcher
RUN git clone https://github.com/api7/lua-resty-ipmatcher.git && \
    cd lua-resty-ipmatcher && \
    luarocks make --tree=/root/.luarocks --lua-dir=$LUA_DIR && \
    cd .. && rm -rf lua-resty-ipmatcher

# lua-system-constants
RUN git clone https://github.com/Kong/lua-system-constants.git && \
    cd lua-system-constants && \
    luarocks make --tree=/root/.luarocks --lua-dir=$LUA_DIR && \
    cd .. && rm -rf lua-system-constants


# Install Lua dependencies
WORKDIR /usr/local/kong
RUN luarocks install --tree=/root/.luarocks lua-cjson 2.1.0.10 --lua-dir=$LUA_DIR && \
    luarocks install --tree=/root/.luarocks luafilesystem --lua-dir=$LUA_DIR && \
    luarocks install --tree=/root/.luarocks luasocket --lua-dir=$LUA_DIR && \
    luarocks install --tree=/root/.luarocks lua-resty-http --lua-dir=$LUA_DIR && \
    luarocks install --tree=/root/.luarocks lua-resty-openssl --lua-dir=$LUA_DIR && \
    luarocks install --tree=/root/.luarocks penlight --lua-dir=$LUA_DIR && \
    luarocks install --tree=/root/.luarocks lua-protobuf \
        LUA_INCDIR=$LUA_DIR/include/luajit-2.1 LUA_LIBDIR=$LUA_DIR/lib && \
    luarocks install --tree=/root/.luarocks lua-resty-ljsonschema --lua-dir=$LUA_DIR && \
    luarocks install --tree=/root/.luarocks version --lua-dir=$LUA_DIR

RUN mkdir -p /usr/local/lib/lua/5.1 && \
    cp /root/.luarocks/lib/lua/5.1/pb.so /usr/local/lib/lua/5.1/

# === STAGE 2: Runtime ===
FROM registry.access.redhat.com/ubi8/ubi-minimal

LABEL maintainer="your-name@your-org.com"

RUN microdnf install -y openssl libyaml hostname perl shadow-utils && microdnf clean all

RUN useradd --system kong && \
    mkdir -p \
      /usr/local/kong/logs \
      /usr/local/kong/pids \
      /usr/local/kong/tmp \
      /usr/local/kong/.cache \
      /usr/local/kong/conf && \
    touch \
      /usr/local/kong/logs/error.log \
      /usr/local/kong/logs/access.log \
      /usr/local/kong/logs/admin_error.log \
      /usr/local/kong/logs/admin_access.log \
      /usr/local/kong/.ca_combined && \
    chmod -R 777 /usr/local/kong && \
    chmod 666 /usr/local/kong/.ca_combined

COPY --from=builder /usr/local/kong /usr/local/kong
COPY --from=builder /usr/local/openresty /usr/local/openresty
COPY --from=builder /root/.luarocks/share/lua/5.1/ /usr/local/share/lua/5.1/
COPY --from=builder /root/.luarocks/lib/lua/5.1/ /usr/local/lib/lua/5.1/
COPY --from=builder /usr/local/lib/lua/5.1/pb.so /usr/local/lib/lua/5.1/

ENV PATH="/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin:/usr/local/kong/bin:$PATH"
ENV LUA_PATH="/usr/local/share/lua/5.1/?.lua;/usr/local/share/lua/5.1/?/init.lua;;"
ENV LUA_CPATH="/usr/local/lib/lua/5.1/?.so;;"
ENV KONG_PREFIX=/usr/local/kong
ENV KONG_CONF=/usr/local/kong/kong.conf

WORKDIR /usr/local/kong

CMD ["kong", "start"]

