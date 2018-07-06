---
title: Installation
table_of_contents: true
---

# Installation

## Prerequisites

To run the Snap Enterprise Proxy, you will need:

* A server running Ubuntu 16.04 LTS on AMD64.
* The ability (e.g. firewall rules) for the server to initiate network
  connections to https://api.snapcraft.io,
  https://public.apps.ubuntu.com, and https://login.ubuntu.com.
* A domain name for the server.
* A PostgreSQL instance, and credentials for a user with `CREATEROLE`
  privileges, and either `CREATEDB` or a pre-created database with `CREATE`
  privileges on it for that user.  The snap-store-proxy snap will create
  "snapauth" and "snaprevs" users, so you will need to ensure that any
  relevant ACLs permit those.
* An RSA key pair to register the snap-store-proxy identity (these can be
  generated for you with: `snap-store-proxy generate-keys`).

## Getting started

First, if your network requires an HTTPS proxy to get to the above
domains, you must first configure snapd on the installation server to
use that HTTPS proxy in order to be able to install the snap-store-proxy snap
package.

Do this by adding the appropriate environment variables (`http_proxy`,
`https_proxy`) to the server’s `/etc/environment` file, and restarting
snapd:

    sudo systemctl restart snapd

!!! NOTE:
    While the Snap Enterprise Proxy is in beta the snap can be installed using:

        sudo snap install snap-store-proxy --beta

Installing the stable release of the Snap Enterprise Proxy is as simple as:

    sudo snap install snap-store-proxy

This will install the snap, which provides a collection of systemd
services, and the `snap-proxy` CLI tool to control the proxy.

## Domain configuration

The enterprise proxy will require a domain or IP address to be set
for the configuration and access by other devices.

    sudo snap-proxy config proxy.domain="<domain>"

This can be done after the database is created, but is required
before registration can succeed.

## Database

To initially configure the Snap Enterprise Proxy, you will need a domain name
for it and either a created database or a connection string including a user
with CREATEDB permissions.

This will set up the proxy to run and listen on all interfaces on port
443 (with a redirect from 80).

Once the database is configured, your proxy should now be
ready to be [registered](register.html).

### Prepared database

If the database is already prepared, set the connection string.

    sudo snap-proxy config \
        proxy.db.connection="postgresql://user:password@host:port/db"

This will require a user with CREATEROLE permission but does not require CREATEDB
permissions.

### Creating a database

There is a convenience option that will create and configure the database
automatically.

The `create-database` command can create a database with a connection string
including a user that has the appropriate permissions (CREATEDB).

    sudo snap-proxy create-database \
        "postgresql://user:password@host:port/db"

This will create and migrate the database.

## Network connectivity

You can check that the Proxy can access all the network locations it
needs to with:

    snap-proxy check-connections

If you require an HTTPS proxy, you can configure the proxy to use that
with:

    sudo snap-proxy config proxy.https.proxy=myproxy.internal:3128


## Running multiple proxies

You can run multiple instances of the proxy for HA, provided by simple
round-robin DNS. All instances need to have the same configuration,
using your normal configuration managment system. Once a key pair has
been registered, it will not need registering on other instances.

!!! NOTE:
    The download caching will currently be less efficient when
    running multiple instances, as the instances are not aware of each
    other. Support for shared caching is planned for later releases of the
    Snap Enterprise Proxy.
