# Kong Gateway 3.6.1 OSS Build for s390x (LinuxONE)

This project documents the process and findings around building Kong Gateway OSS 3.6.1 from source on the IBM LinuxONE (s390x) architecture.

---

## ✅ Successful Work Completed

### 1. Environment Setup
- **Platform**: RHEL 8 on IBM LinuxONE (s390x)
- **Base Image**: UBI 8
- Installed:
  - `gcc`, `make`, `git`, `wget`, `curl`
  - `luarocks`, `openresty`, `protobuf`, `openssl`, `zlib`, `pcre`
  - Lua 5.1, all required Lua modules

### 2. Lua Modules Installed (via LuaRocks)
- `lua-cjson`
- `lua-resty-http`
- `lua-resty-jit-uuid`
- `lua-resty-timer-ng`
- `lua-resty-ipmatcher`
- `lua-resty-openssl`
- `penlight`
- `protobuf` (native `pb.so` compiled)

### 3. Custom Dockerfile
- Multi-stage build process for Kong OSS 3.6.1
- Image pushed to: `quay.io/tonyfieit75/kong-oss:3.6.1-s390x-1`

### 4. Runtime Tests
- Verified:
  - `kong version`
  - `kong health`
  - Container launched successfully with basic config

---

## ⚠️ Limitations Encountered

### ❌ Proprietary OpenResty Dependencies
- Kong 3.6.1 relies on a custom fork of OpenResty containing internal patches:
  - `ngx_http_lua_module`, `lua-cjson`, others
  - Not included in OSS source

### ❌ Private Kong CI Resources
- Kong’s OpenResty build is not public.
- Requires access to Kong Enterprise GitHub or Partner CI builds

### ❌ Missing Runtime Directives
- Errors during `kong start`:
  ```
  nginx: [emerg] unknown directive "lmdb_environment_path"
  nginx: [emerg] unknown directive "kong_ssl"
  ```

---

## 🧾 Summary

- Kong OSS 3.6.1 can be built and launched on s390x with open-source resources.
- However, full functionality is **blocked** due to missing proprietary OpenResty patches and directives.
- A full working build **requires Enterprise access**.

---

## 📌 Recommendation

To proceed:
- Request access to Kong’s internal OpenResty repository or Enterprise source tree.
- Engage with Kong through their **partner support** or official channels.