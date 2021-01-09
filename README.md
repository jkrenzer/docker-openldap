# The `mlan/openldap` repository

![travis-ci test](https://img.shields.io/travis/mlan/docker-openldap.svg?label=build&style=popout-square&logo=travis)
![docker build](https://img.shields.io/docker/build/mlan/openldap.svg?label=build&style=popout-square&logo=docker)
![image Size](https://img.shields.io/docker/image-size/mlan/openldap.svg?label=size&style=popout-square&logo=docker)
![docker pulls](https://img.shields.io/docker/pulls/mlan/openldap.svg?label=pulls&style=popout-square&logo=docker)
![docker stars](https://img.shields.io/docker/stars/mlan/openldap.svg?label=stars&style=popout-square&logo=docker)
![github stars](https://img.shields.io/github/stars/mlan/docker-openldap.svg?label=stars&style=popout-square&logo=github)

This (non official) Docker image contains an [OpenLDAP](https://www.openldap.org/) directory server. A directory server typically holds user login credentials, postal and e-mail addresses and similar pieces of information. This image help integrate a directory server with other dockerized services.

## Features

- [OpenLDAP](https://www.openldap.org/) directory server
- Database creation or recreation during initial run using `LDIF` backup files
- Database creation from template `LDIF` files with parameters from environment variables
- Log directed to docker daemon with configurable level
- Accepts read only (RO) mounted database file systems
- Built in LDAP client tools helping managing the databases
- Built in `LDIF` filters helping initiating and managing the databases
- [Unix domain (IPC) socket](https://en.wikipedia.org/wiki/Unix_domain_socket) (ldapi:///) support
- Configurable database paths, helping host volume management
- Configurable run-as-user `uid` and `gid`
- Small image size based on [Alpine Linux](https://alpinelinux.org/)
- Health check

## Tags

The MAJOR.MINOR.PATCH [SemVer](https://semver.org/) is used. In addition to the three number version number you can use two or one number versions numbers, which refers to the latest version of the sub series. The tag `latest` references the build based on the latest commit to the repository.

To exemplify the usage of version tags, lets assume that the latest version is `1.2.3`. In this case `latest`, `1.2.3`, `1.2` and `1` all identify the same image.

# Usage

A [Directory Server Agent (DSA)](https://en.wikipedia.org/wiki/Directory_System_Agent) is used to store, organize and present data in a key-value type format. Typically, directories are optimized for lookups, searches, and read operations over write operations, so they function extremely well for data that is referenced often but changes infrequently.

The [mlan/openldap](https://github.com/mlan/docker-openldap) repository provides such an OpenLDAP directory server in a docker container.

## Directory service

The [Lightweight Directory Access Protocol (LDAP)](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol), is an open protocol used to store and retrieve data from a hierarchical directory structure. Commonly used to store information about an organization and its assets and users.

[OpenLDAP](https://www.openldap.org/) is a cross platform LDAP based directory service. [Active Directory (AD)](https://en.wikipedia.org/wiki/Active_Directory) is a [directory service](https://en.wikipedia.org/wiki/Directory_service) developed by [Microsoft](https://en.wikipedia.org/wiki/Microsoft) for [Windows domain](https://en.wikipedia.org/wiki/Windows_domain) networks which also uses LDAP.

### Directory entries

The user data itself is mainly stored in elements called attributes. Attributes are basically key-value pairs. Unlike in some other systems, the keys have predefined names which are dictated by the `objectClass` selected for an entry. An entry is basically a collection of attributes under a name used to describe something; a user for example.

```
dn: uid=demo,ou=users,dc=example,dc=com
objectClass: inetOrgPerson
cn: demo
sn: demo
uid: demo
```

### Directory queries

An [LDAP query](https://ldapwiki.com/wiki/LDAP%20Query%20Basic%20Examples) is a command that asks a directory service for some information. For instance, if you like to see the record of a particular user, you can submit a query that looks like this:

```lisp
(&(objectClass=inetOrgPerson)(uid=demo))
```

### Directory authentication

There are two options for LDAP authentication; simple and SASL (Simple Authentication and Security Layer).

Simple authentication allows for three possible authentication mechanisms: Anonymous authentication, grants client anonymous status to LDAP. Unauthenticated authentication, for logging purposes only, should not grant access to a client. Name and password authentication, grants access to the server based on the credentials supplied.

SASL authentication binds the LDAP server to another authentication mechanism, like Unix passwd, which is used when using the buitin client commands.

It’s important to note that LDAP passes messages in clear text by default. You need to add TLS encryption or similar to keep your usernames and passwords safe.

### Directory socket `ldapi:///` scheme

When you have direct access to the container you can use the built in LDAP client tools, using the Unix socket `ldapi:///`, to query and manage the directory.

Most directories are configured to allow the root user, via SASL, full access to the database. So on the docker host you can issue this command to query the directory.

```sh
docker exec auth ldapsearch "(&(objectClass=inetOrgPerson)(cn=demo))"
```

### Directory network `ldap:///` scheme

Other containers most often communicate with the directory service over the network. The standard LDAP port is 389. For secure LDAPS the 636 port is used.

Most directories are configured to allow anonymous queries. So remote queries can look like this.

```sh
ldapsearch -H ldap://auth:389/ -b dc=example,dc=com "(&(objectClass=inetOrgPerson)(cn=demo))"
```

## Docker Compose example

An example of how to define an OpenLDAP directory service using docker compose is given below. Other services that also use the `backend` network will be able to access the directory service.

```yaml
version: '3'
services:
  auth:
    image: mlan/openldap
    networks:
      - backend
    command: --root-cn ${LDAPROOT_CN-admin} --root-pw ${LDAPROOT_PW-secret}
    environment:
      - LDAPBASE=${LDAPBASE-dc=example,dc=com}
      - LDAPLOGLEVEL=${LDAPLOGLEVEL-parse}
    volumes:
      - auth:/srv
      - /etc/localtime:/etc/localtime:ro        # Use host timezone

networks:
  backend:

volumes:
  auth:
```

## Demo

This repository contains a [demo](demo) directory which hold the [docker-compose.yml](demo/docker-compose.yml) file as well as a [Makefile](demo/Makefile) which might come handy. Start by cloning the [github](https://github.com/mlan/docker-openldap) repository.

```bash
git clone https://github.com/mlan/docker-openldap.git
```

From within the [demo](demo) directory you can start the container, and configure the directory database to hold a demo user, by typing:

```bash
make init
```

Now you can send some queries to the directory service you just created. First we can try using the Unix socket scheme:

```sh
make auth-show_data
```

Which translates to:

```sh
docker-compose exec auth ldapsearch
```

We can also send a query over the network:

```sh
make auth-show_contexts
```

Which translates to:

```sh
ldapsearch -H ldap://172.31.0.2/ -xLLL -s base namingContexts
```

You can view the directory database by using a web interface on the URL [`http://localhost:8001`](http://localhost:8008) after you have logged in with the DN: `cn=admin,dc=example,dc=com` and password: `secret`.

```bash
make auth-gui-up
```

When you are done testing you can destroy the test containers by typing

```bash
make destroy
```

## Persistent storage

It is often advantageous to keep the databases on a separate volume instead of within the container itself. The volume survives container destruction and can be synced between hosts, for example. To simplify volume management the databases have been consolidated under `/srv`. The path to the configuration and data databases within the container
are `DOCKER_DB0_VOL=/srv/conf` and `DOCKER_DB1_VOL=/srv/data` respectively. 

Arranging persistent storage can be as easy as typing:
```bash
docker run -d --name auth -v auth:/srv mlan/openldap
```

You can also see the [docker compose example](#docker-compose-example) on how persistent storage can be arranged.

## Startup procedure

The startup sequence of the `mlan/openldap` container consists of two phases: 1) If databases are not available, try to recreate them from [LDIF](https://en.wikipedia.org/wiki/LDAP_Data_Interchange_Format) backup files. 2) Start the [Stand-alone LDAP Daemon (slapd)](https://en.wikipedia.org/wiki/Slapd). The config and directory databases are kept under `/srv/conf` and `/srv/data` respectively. They are often stored using a single docker volume mounted at `/srv`, so that the container can be started using a command like this


```sh
docker run -d --name auth -v auth:/srv mlan/openldap
```

Or using [docker compose](https://docs.docker.com/compose/) and a `docker-compose.yml` file

```yaml
version: '3'
services:
  auth:
    image: mlan/openldap
    volumes:
      - auth:/srv
volumes:
  auth:
```

The `auth` service is fired up using

```sh
docker-compose up -d
```

## Recreate databases

At first, during container startup, valid config and directory databases are checked for. If they are found, the first phase is complete. Second the [Stand-alone LDAP Daemon (slapd)](https://en.wikipedia.org/wiki/Slapd) is started.

In the event that there is no databases, an attempt is made to recreate them from backup [LDIF](https://en.wikipedia.org/wiki/LDAP_Data_Interchange_Format) files. First the config database, number __0__, is recreated and second the directory database, number __1__, is recreated. If no backup files are found the config database is created from a sample LDIF file, but the directory database will not be created.

The file system paths that are searched for backup LDIF files for the config database 0 is defied by `DOCKER_SLAPADD0_PATH="/ldif/0:/0.ldif:/etc/openldap/slapd.ldif"`. And similarly for the directory database 1 the paths searched are `DOCKER_SLAPADD1_PATH="/ldif/1:/1.ldif"`. There are several methods by which backup files can be made available to the container including; bind mount, docker configs, docker secrets and docker cp. If the backup files are [gzip](https://en.wikipedia.org/wiki/Gzip) compressed they will automatically be decompressed.

For example; to use the backup file `cfg.example.com.ldif`, to recreate the config database, it can be bind mounted into the container

```sh
docker run -d --name auth -v $(pwd)/ldif/0/cfg.example.com.ldif:/etc/openldap/slapd.ldif mlan/openldap
```

Similarly using docker-compose, both database 0 and 1 can be bind mounted

```yaml
version: '3'
services:
  auth:
    image: mlan/openldap
    volumes:
      - auth:/srv
      - ./ldif:/ldif
```

Docker secrets can be used like this

```yaml
version: '3'
services:
  auth:
    image: mlan/openldap
    volumes:
      - auth:/srv
    secrets:
      - 0.ldif
secrets:
  0.ldif:
    file: ./ldif/0/cfg.example.com.ldif
```

Alternatively we can create the container, copy the LDIF file to it and then start the container

```sh
docker create --name auth mlan/openldap
docker cp ldif/. auth:/ldif
docker start auth
```

If no LDIF files are provided the builtin sample configuration file `/etc/openldap/slapd.ldif` will be used. It defines the root DN to `cn=admin,dc=example,dc=com` with password `secret`.

## Create databases

A new configuration database can be created by modifying a sample LDIF file like the build in `/etc/openldap/slapd.ldif`. Many of the parameters in this file might apply to a new configuration with the exception of the database BASE, root DN and its password. LDIF filters are available to update these parameters. The filters can be controlled by both commands arguments and environment variables. 

The following creates a new configuration database based on the builtin sample with updated BASE, root DN and password.

```sh
docker run -d --name auth -e LDAPBASE=dc=sample,dc=org mlan/openldap --root-cn manager --root-pw challenge
```

Similarly with docker compose but now using the file `ldif/0/cfg.example.com.ldif` as a template.

```yaml
version: '3'
services:
  auth:
    image: mlan/openldap
    command: --root-cn admin --root-pw secret
    environment:
      - LDAPBASE=dc=example,dc=com
    configs:
      - 0.ldif
configs:
  0.ldif:
    file: ./ldif/0/cfg.example.com.ldif
```

Please see section [environment variables](#environment-variables) for a description of available commands arguments and environment variables. 

## Manage databases

Once the container is running the databases can be managed using the builtin OpenLDAP client utilities (as well as any LDAP client).

If, for example, you have the directory database in the file `ldif/sample/dit.my-domain.org.ldif` you can add it by using the builtin client like this:


```sh
docker exec -i auth ldapadd < ldif/sample/dit.my-domain.org.ldif
```

If the container was started using docker-compose, use this command to add `ldif/1/dit.example.com.ldif`:

```sh
docker-compose exec -T auth ldapadd < ldif/1/dit.example.com.ldif
```

We list the LDAP client commands here for convenience.

| Command                                                      | Example                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [ldapadd](https://www.openldap.net/software/man.cgi?query=ldapadd) | docker-compose exec -T auth ldapadd < add.ldif               |
| [ldapcompare](https://www.openldap.net/software/man.cgi?query=ldapcompare) | docker exec auth ldapcompare cn=user,dc=example,dc=com sn:user |
| [ldapdelete](https://www.openldap.net/software/man.cgi?query=ldapdelete) | docker exec auth ldapdelete cn=user,dc=example,dc=com        |
| [ldapexop](https://www.openldap.net/software/man.cgi?query=ldapexop) | docker exec auth ldapexop whoami                             |
| [ldapmodify](https://www.openldap.net/software/man.cgi?query=ldapmodify) | docker exec -i auth ldapmodify < modify.ldif                 |
| [ldapmodrdn](https://www.openldap.net/software/man.cgi?query=ldapmodrdn) | docker exec -i auth ldapmodrdn < modrdn.ldif                 |
| [ldappasswd](https://www.openldap.net/software/man.cgi?query=ldappasswd) | docker exec auth ldappasswd -SAWD cn=user,dc=example,dc=com  |
| [ldapsearch](https://www.openldap.net/software/man.cgi?query=ldapsearch) | docker exec auth ldapsearch                                  |
| [ldapurl](https://www.openldap.net/software/man.cgi?query=ldapurl) | docker exec auth ldapurl                                     |
| [ldapwhoami](https://www.openldap.net/software/man.cgi?query=ldapwhoami) | docker exec auth ldapwhoami -WD cn=user,dc=example,dc=com    |


## Environment variables

When you start an `mlan/openldap` container, you can adjust its configuration by passing one or more environment variables on the `docker run` command line. Once the instance has been run for the first time and the config and users databases has been created these variables will _not_ have any effect. Any existing databases will always be left untouched on container startup.

The available commands arguments and environment variables are listed below.

| Argument  | Environment, -e | Example           |
| --------- | --------------- | ----------------- |
| --base    | LDAPBASE        | dc=example,dc=com |
| --root-cn | LDAPROOT_CN     | admin             |
| --root-pw | LDAPROOT_PW     | secret            |
| --runas   | LDAPRUNAS       | 1001:1002         |
| --debug   | LDAPDEBUG       | stats             |

#### `LDAPBASE`

The `LDAPBASE` environment variable serves two purposes. Firstly it servers as a LDIF filter and update the directory database suffix during [database creation](#create-databases). Secondly it defines the default search base for the builtin [LDAP client utilities](#manage-databases). 

When creating databases, a sample file can be used together with filters to update the database BASE, root DN and its password. The `LDAPBASE` environment variable update the `olcSuffix` of the database when it is created. Example usage: `LDAPBASE=dc=example,dc=com`. Often you also want to update the `LDAPROOT_CN` and `LDAPROOT_PW` , see below for their descriptions.

As mentioned above, when the `LDAPBASE` environment variable is set you can skip specifying the search base with the -b option.

#### `LDAPROOT_CN`

The `LDAPROOT_CN` variable updates the root _distinguished name_ in the config database, using this _common name_ as a base. To have any effect both `LDAPROOT_CN` and `LDAPBASE` need to be set to non-empty strings.
Example usage: `LDAPBASE=dc=example,dc=com` and `LDAPROOT_CN=admin` will set/change the the root _distinguished name_ (DN) to `cn=admin,dc=example,dc=com`.

#### `LDAPROOT_PW`

The `LDAPROOT_PW` updates root password, during the database creation. The password can be given in clear text or its hashed equivalent. Example: `LDAPROOT_PW={SSHA}KirjzsjgMvjqBWNNof7ujKhwAZBfXmw3` (generated using `slappasswd -s secret`).

The root DN of the builtin sample LDIF file is `cn=admin,dc=example,dc=com`. and its password is `secret`.


#### `LDAPDEBUG`

Controls the debug messages sent to stdout. The default value is `LDAPDEBUG=none`. Start the container with `LDAPDEBUG=stats` if you want to see more messages. Please refer to
[slapd configuration](https://www.openldap.org/doc/admin24/slapdconfig.html)
for all options available. Normal logging, including dynamic control via `olcLogLevel`, does _not_ work since there is no `syslog` daemon running inside the container.

#### `LDAPRUNAS`

The `mlan/openldap` container run the `slapd` ([Stand-alone LDAP Daemon](https://www.openldap.org/software/man.cgi?query=slapd)) as root. You can set this explicitly by defining `LDAPRUNAS=root.`

Should you want to run the daemon as a no-privileged user within the container, set `LDAPRUNAS=ldap`.

Sometimes you want the daemon and its files to be run by and owed by a specific user and group ID. If so, use `LDAPRUNAS=<uid>:<gid>` or if the `uid` and `gid` is equal `LDAPRUNAS=<uid>:` Example usage: `LDAPRUNAS=120:127` which will run the daemon and have its files owned by user `uid` 120 and group `gid` 127. This can be useful when bind mounting volumes on the docker host.

# Knowledge base

Here some topics relevant for directory servers are presented.

## Recreate the host database

This use case relies on that config and users data are available in an preexisting OpenLDAP server.

First, create backup `LDIF` files of the host OpenLDAP server databases:

```bash
sudo slapcat -n0 -o ldif-wrap=no -l ldif/0/cfg.host.ldif
sudo slapcat -n1 -o ldif-wrap=no -l ldif/1/dit.host.ldif
```

Second make these backup `LDIF` files available to a container and run it. One option is to bind mount the host directory the files reside in:

```bash
docker run -d --name openldap -p 389:389 -v $(pwd)/ldif:/ldif mlan/openldap
```
Another is to first copying the host directory to a created but not running container and then run it:
```bash
docker create --name auth -p 389:389 mlan/openldap
docker cp ldif/. auth:/ldif
docker start auth
```
Since you might have a directory server running on the same host as you are running docker containers, you might need to change the port of you container to allow it to start.

# Implementation

## Database locations

Typical paths are `DOCKER_DB0_DIR=/etc/openldap/slapd.d` and `DOCKER_DB1_DIR=/var/lib/openldap/openldap-data`.
Here these paths are symlinked to `DOCKER_DB0_VOL=/srv/conf` and `DOCKER_DB1_VOL=/srv/data` respectively.

When the container is started and the directories `/srv/conf` and `/srv/data`
are mounted read only (RO), they are copied to `/tmp/conf` and `/tmp/data`, as defined by `DOCKER_RWCOPY_DIR=/tmp`, and the symlinks in `/etc/openldap/slapd.d` and `/var/lib/openldap/openldap-data` are changed accordingly.
This directory path is used to determine the paths where copies `DOCKER_DB0_VOL` and `DOCKER_DB1_VOL` are placed if they are mounted read only. If you wish to disable this feature set `DOCKER_RWCOPY_DIR=` to empty.
