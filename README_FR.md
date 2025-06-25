# Compilation de Kong Gateway 3.6.1 OSS pour s390x (LinuxONE)

Ce projet documente le processus et les résultats liés à la compilation de Kong Gateway OSS 3.6.1 à partir du code source sur l'architecture IBM LinuxONE (s390x).

---

## ✅ Travaux Réussis

### 1. Configuration de l’Environnement
- **Plateforme** : RHEL 8 sur IBM LinuxONE (s390x)
- **Image de base** : UBI 8
- Paquets installés :
  - `gcc`, `make`, `git`, `wget`, `curl`
  - `luarocks`, `openresty`, `protobuf`, `openssl`, `zlib`, `pcre`
  - Lua 5.1 et tous les modules Lua requis

### 2. Modules Lua installés (via LuaRocks)
- `lua-cjson`
- `lua-resty-http`
- `lua-resty-jit-uuid`
- `lua-resty-timer-ng`
- `lua-resty-ipmatcher`
- `lua-resty-openssl`
- `penlight`
- `protobuf` (compilé en natif avec `pb.so`)

### 3. Dockerfile personnalisé
- Processus de compilation multi-étapes pour Kong OSS 3.6.1
- Image poussée sur : `quay.io/tonyfieit75/kong-oss:3.6.1-s390x-1`

### 4. Tests d’exécution
- Vérifications :
  - `kong version`
  - `kong health`
  - Conteneur lancé avec succès avec configuration de base

---

## ⚠️ Limitations Rencontrées

### ❌ Dépendances OpenResty Propriétaires
- Kong 3.6.1 dépend d’un fork propriétaire de OpenResty incluant des correctifs internes :
  - `ngx_http_lua_module`, `lua-cjson`, etc.
  - Non inclus dans les sources OSS publiques

### ❌ Ressources CI Privées de Kong
- La version OpenResty utilisée par Kong n’est pas publique.
- Accès requis via un abonnement Enterprise ou programme partenaire

### ❌ Directives Manquantes à l’Exécution
- Erreurs rencontrées avec `kong start` :
  ```
  nginx: [emerg] unknown directive "lmdb_environment_path"
  nginx: [emerg] unknown directive "kong_ssl"
  ```

---

## 🧾 Résumé

- Kong OSS 3.6.1 peut être compilé et lancé sur s390x avec des ressources open source.
- Toutefois, les fonctionnalités complètes sont **bloquées** en raison de l’absence de correctifs OpenResty propriétaires.
- Une compilation fonctionnelle complète **nécessite un accès Enterprise**.

---

## 📌 Recommandation

Pour aller plus loin :
- Demander un accès au dépôt privé OpenResty de Kong ou au code source Enterprise.
- Contacter le support partenaire officiel de Kong.