# 2.0.0

- [docker](Dockerfile) Now build using alpine:3.12 (openldap 2.4.50).
- [demo](demo) Updated demo.
- [test](test) Code refactoring.
- [docker](src/docker/bin/docker-entrypoint.sh) Code refactoring.
- [openldap](src/openldap/bin/openldap-common.sh) _BREAKING!_ New initialization procedure.
- [openldap](src/openldap/bin/openldap-common.sh) _BREAKING!_ Utility script `ldap` functionality abandoned, use ldap client tools directly instead.
- [openldap](src/openldap/bin/openldap-common.sh) Now default ldapi socket is used.
- [openldap](src/openldap/config/slapd.ldif) _BREAKING!_ Now no default domain component entry, `dcObject`, is created.
- [docker](README.md) Reworked documentation.

# 1.1.0

- [repo](src) Put source code in the dir `src` and tests in `test`.

# 1.0.2

- Now build using alpine:3.11.
- Cosmetic changes.

# 1.0.1

- Now build using alpine:3.10.

# 1.0.0

- Log directed to docker daemon with configurable level.
- Accepts read only (RO) mounted database file systems.
- Unix domain (IPC) socket support.
- Configurable database paths, helping host volume management.
- Code refactoring.
