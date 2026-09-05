# ubixcore-php

The PHP runtime image for projects built on [uBixCore](https://github.com/ubixsys/ubixcore):
nginx, PHP-FPM, the memcache and memcached extensions, Composer, and supervisord
on Alpine Linux. Multi-arch (amd64, arm64), runs as an unprivileged user,
listens on 8080.

```dockerfile
FROM ghcr.io/ubixsys/ubixcore-php:8.5
COPY --chown=www:www . /web
RUN composer install --no-dev --no-interaction
```

That is the whole contract: put your project under `/web`, and nginx serves
`/web/root` (uBixCore hosts symlink or copy `public/` there). Health probe:
`GET /fpm-ping` on 8080.

## Tags

| Tag | PHP | Alpine | Notes |
|---|---|---|---|
| `8.5`, `latest` | 8.5 | 3.23 | default for new projects |
| `8.4` | 8.4 | 3.22 | supported while uBixCore stays 8.4-compatible |
| `<version>-<yyyymmdd>` | | | immutable build of that day, for pinning |

Each PHP line is one directory (`8.5/`, `8.4/`) holding its `Dockerfile` and
`config/` (nginx, PHP-FPM pool, `php.ini` overrides, supervisord, ulimits).

## What is inside

Extensions: ctype, curl, dom, fileinfo, gd, iconv, intl, json, mbstring, mysqli,
opcache, openssl, pdo, pdo_mysql, pecl-memcache, pecl-memcached, phar, session,
simplexml, tokenizer, xml, xmlreader, xmlwriter, zip, zlib. Timezone is UTC;
set `TZ` or a `php.ini` override in your image if you need another.

PHP-FPM listens on `127.0.0.1:9000` (`pm = ondemand`), nginx on `8080`.
Logs go to the container's stdout/stderr. `/fpm-status` and `/fpm-ping` are
reachable from localhost and RFC 1918 `10.0.0.0/8` only.

## Building locally

```bash
docker build -t ubixcore-php:8.5 8.5/
docker run --rm -p 8080:8080 ubixcore-php:8.5
```

## Releases

Every push builds each changed PHP line to the project's own registry as
`<version>-<branch>`. Merging to `main` publishes `<version>`,
`<version>-<yyyymmdd>` and, for the newest line, `latest` to
`ghcr.io/ubixsys/ubixcore-php`. Pins (Alpine minor, `phpNN` package family)
are bumped on purpose, one per commit, and noted in `CHANGELOG.md`.

Secrets for the publish step live in [uBixVault](https://github.com/cwolsen7905/ubixvault),
never in the repository or CI variables beyond the read-only Vault token.

License: BSD-3-Clause.
