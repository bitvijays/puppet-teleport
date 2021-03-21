# puppet-teleport

[![Build Status](https://travis-ci.org/jaxxstorm/puppet-teleport.svg?branch=master)](https://travis-ci.org/jaxxstorm/puppet-teleport)

#### Table of Contents

1. [Module Description - What the module does and why it is useful](#module-description)
2. [Setup - The basics of getting started with puppet-teleport](#setup)
    * [What puppet-teleport affects](#what-puppet-teleport-affects)
    * [Beginning with puppet-teleport](#beginning-with-puppet-teleport)
3. [Usage - Configuration options and additional functionality](#usage)
4. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Development - Guide for contributing to the module](#development)


## Module description

This module will download, install and configure [Teleport](https://github.com/gravitational/teleport) a cluster SSH tool created by [Gravitational](https://gravitational.com).

For more information about Teleport, see the [documentation](http://gravitational.com/teleport/docs/quickstart/)

## Setup

### What puppet-teleport affects

puppet-teleport will:

  * Download the required golang binary from the teleport releases page and install it
  * Create a service/init script on your OS to start teleport
  * Configure the yaml config file and set up the required role

### Beginning with puppet-teleport

By default, puppet-teleport will configure teleport with a "node" role. Simple include the teleport module like so

```puppet
  include ::teleport
```

Teleport has multiple [roles](http://gravitational.com/teleport/docs/architecture/#high-level-overview) which are run from the same binary. In order to configure the use of these roles, you need to configure them in the yaml, and these can be done as parameters to the main teleport class. An example of this might be:


```puppet
  class { '::teleport':
    proxy_enable       => true,
    proxy_listen_addr  => '0.0.0.0',
  }
```

## Usage

### I just want to install teleport, what's the minimum I need?

```puppet
  include ::teleport
```

### I want to install all the teleport components, what do I need?

```puppet
  class { '::teleport':
    auth_enable  => true,
    proxy_enable => true,
  }
```

### I want to auth to another auth server, what do I need?

```puppet

  class { '::teleport':
    auth_servers => ['192.168.4.10', 192.168.4.11'],
  }
```

### Setup a Teleport Server

- auth_token: Token used to connect the clients

```puppet
  class { '::teleport':
    version               => '5.1.0',
    auth_enable           => true,
    proxy_enable          => true,
    auth_listen_addr      => '0.0.0.0',
    proxy_web_listen_addr => '0.0.0.0',
    auth_service_tokens   => [ 'node:pNVNrTIupUieGWGR0vz5LNOxUaNbgIgjsEZaIAxxGkHlz']
  }
```

### Setup a Teleport Client

- auth_servers is the IP Address of Teleport Server
- auth_token is the token used to connect the clients to the Teleport Server 

```puppet
  class { '::teleport':
    version      => '5.1.0',
    auth_token   => 'pNVNrTIupUieGWGR0vz5LNOxUaNbgIgjsEZaIAxxGkHlz',
    auth_servers => ['192.168.X.X']
  }
```

## Reference

### Classes

#### Public Classes
  * [`teleport`](#teleport): Installs and configured teleport in your environment

#### Private Classes
  * [`teleport::install`]: Downloads the teleport binary and installs it in your env
  * [`teleport::config`]: Configure the service and the teleport config file
  * [`teleport::service`]: Manage the teleport service


### `teleport`

#### Parameters

##### `version` [String]

Specifies the version of teleport to download

##### `archive_path` [String]

Where to download the teleport tarball

##### `extract_path` [String]

Directory to extract teleport

##### `bin_dir` [String]

Where to symlink teleport binaries

##### `assets_dir` [Bool]

Where to sylink the teleport web assets

##### `nodename` [String]

Teleport nodename. Default: `$::fqdn`

#### `auth_type` [String]

default authentication type. possible values are 'local', 'oidc' and 'saml'
only local authentication (Teleport's own user DB) is supported in the open
source version
Defaults to 'local'

#### `auth_second_factor` [String]

Second_factor can be off, otp, or u2f
Defaults to 'otp'

#### `auth_u2f_app_id` [String]

app_id must point to the URL of the Teleport Web UI (proxy) accessible
by the end users. Only used if auth_second_factor is set to 'u2f'
Defaults to 'https://localhost:3080'

#### `auth_u2f_facets` [Array]

facets must list all proxy servers if there are more than one deployed. Type array.
Only used if auth_second_factor is set to 'u2f'
Defaults to ['https://localhost:3080']

##### `data_dir` [String]

Teleport data directory.

##### `auth_token` [String]

The auth token to use when joining the cluster

#### `auth_cluster_name` [String]

Optional "cluster name" is needed when configuring trust between multiple
auth servers. A cluster name is used as part of a signature in certificates
generated by this CA.

By default an automatically generated GUID is used.

IMPORTANT: if you change cluster_name, it will invalidate all generated
certificates and keys (may need to wipe out /var/lib/teleport directory)
Defaults to `undef`

##### `advertise_ip` [String]

When running in NAT'd environments, designates an IP for teleport to advertise.

##### `storage_backend` [String]

Which storage backend to use.

##### `storage_options` [Hash]

Extra options for some storage backends, like DynamoDB.

##### `max_connections` [String]

Configure max connections for teleport

##### `max_users` [String]

Teleport max users

##### `log_dest` [String]

Log destination

##### `log_level` [String]

Log output level. Default: `"ERROR"`

#### `config_path` [String]

Path to config file for teleport. Default: `/etc/teleport.yaml`

#### `auth_servers` [Array]

An array of auth servers to connect to

#### `auth_enable` [Bool]

Whether to start the auth service. Default: `false`

#### `auth_listen_addr` [String]

Address to listen for auth_service

#### `auth_listen_port` [String]

Port to listen on for auth server

#### `auth_service_tokens` [Array]

The provisioning tokens for the auth tokens

#### `ssh_enable` [String]

Whether to start SSH service. Default: `true`

#### `ssh_listen_addr` [String]

Address to listen on for SSH connections. Default: `0.0.0.0`

#### `ssh_listen_port` [String]

Port to listen on for SSH connection

#### `labels` [Hash]

A hash of labels to assign to hosts

#### `ssh_label_commands` [Array]

List of the commands to periodically execute. Their output will be used as node labels.
See "Labeling Nodes" section below for more information.
Defaults to
```
[{
  name    => 'arch',
  command => '[uname, -p]',
  period  => '1h0m0s',
}]
```

#### `ssh_permit_user_env` [Bool]

enables reading ~/.tsh/environment before creating a session. by default
set to false, can be set true here or as a command line flag.
Defaults to false

#### `proxy_enable` [Bool]

Where to start the proxy service. Default. `false`

#### `proxy_listen_addr` [String]

Address to listen on for proxy

#### `proxy_listen_port` [String]

Port to listen on for proxy connection

#### `proxy_tunnel_listen_addr` [String]

Reverse tunnel listening address. An auth server (CA) can establish an
outbound (from behind the firewall) connection to this address.
This will allow users of the outside CA to connect to behind-the-firewall
nodes.
Defaults to '127.0.0.1'

#### `proxy_tunnel_listen_port` [String]

Reverse tunnel listening port.
Defaults to 3024

#### `proxy_web_listen_address` [String]

Port to listen on for web proxy connections

#### `proxy_ssl` [Bool]

Enable or disable SSL support. Default: `false`

#### `proxy_ssl_key` [String]

Path to SSL key for proxy

#### `proxy_ssl_cert` [String]

Path to SSL cert for proxy

#### `init_style` [String]

Which init system to use to start the service.

#### `manage_service` [Bool]

Whether puppet should manage and configure the service

#### `service_ensure` [String]

State of the teleport service (Running/Stopped)

#### `service_enable` [Bool]

Whether the service should be enabled on startup


## Limitations

Currently only works on Linux
