<p><img src="https://code.benco.io/icon-collection/logos/ansible.svg" alt="ansible logo" title="ansible" align="left" height="60" /></p>
<p><img src="https://in-trend.biz/wp-content/uploads/2017/10/nanopool.png" alt="nanopool logo" title="nanopool" align="right" height="60" /></p>

Ansible Role :lock_with_ink_pen: :link: Zecminer
=========
[![Galaxy Role](https://img.shields.io/ansible/role/46755.svg)](https://galaxy.ansible.com/0x0I/zecminer)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/0x0I/ansible-role-zecminer?color=yellow)
[![Downloads](https://img.shields.io/ansible/role/d/46755.svg?color=lightgrey)](https://galaxy.ansible.com/0x0I/zecminer)
[![Build Status](https://travis-ci.org/0x0I/ansible-role-zecminer.svg?branch=master)](https://travis-ci.org/0x0I/ansible-role-zecminer)
[![License: MIT](https://img.shields.io/badge/License-MIT-blueviolet.svg)](https://opensource.org/licenses/MIT)

**Table of Contents**
  - [Supported Platforms](#supported-platforms)
  - [Requirements](#requirements)
  - [Role Variables](#role-variables)
      - [Install](#install)
      - [Config](#config)
      - [Launch](#launch)
      - [Uninstall](#uninstall)
  - [Dependencies](#dependencies)
  - [Example Playbook](#example-playbook)
  - [License](#license)
  - [Author Information](#author-information)

Ansible role that installs and configures EWBF's Zcash Zecminer: a CUDA-based GPU miner for solving the Equihash Proof-of-Work Algorithm.

##### Supported Platforms:
```
* Redhat(CentOS/Fedora)
* Ubuntu
```

Requirements
------------
Requires the `unzip/gtar` utility to be installed on the target host. See ansible `unarchive` module [notes](https://docs.ansible.com/ansible/latest/modules/unarchive_module.html#notes) for details.

Role Variables
--------------
Variables are available and organized according to the following software & machine provisioning stages:
* _install_
* _config_
* _launch_
* _uninstall_

#### Install

`zecminer`can be installed using compressed archives (`.tar`, `.zip`), downloaded and extracted from various sources.

_The following variables can be customized to control various aspects of this installation process, ranging from software version and source location of binaries to the installation directory where they are stored:_

`zecminer_user: <service-user-name>` (**default**: *zecminer*)
- dedicated service user and group used by `zecminer` for privilege separation (see [here](https://www.beyondtrust.com/blog/entry/how-separation-privilege-improves-security) for details)

`install_type: <archive>` (**default**: archive)
- **archive**: currently compatible with **tar** formats, installation of Zecminer via compressed archives results in the direct download of its component binaries, consisting of the `zecminer` mining software.

  **note:** archived installation binaries can be obtained from the official [releases](https://github.com/nanopool/ewbf-miner/releases) site or those generated from development/custom sources.

`install_dir: </path/to/installation/dir>` (**default**: `/opt/zecminer`)
- path on target host where the `zecminer` binaries should be extracted to.

`archive_url: <path-or-url-to-archive>` (**default**: see `defaults/main.yml`)
- address of a compressed **tar or zip** archive containing `zecminer` binaries. This method technically supports installation of any available version of `traefik`. Links to official versions can be found [here](https://github.com/nanopool/ewbf-miner/releases).

`archive_options: <untar-or-unzip-options>` (**default**: `[]`)
- list of additional unarchival arguments to pass to either the `tar` or `unzip` binary at runtime for customizing how the archive is extracted to the designated installation directory. See [man tar](https://linux.die.net/man/1/tar) and [man unzip](https://linux.die.net/man/1/unzip) for available options to specify, respectively.

#### Config

Zecminer supports specification of various options controlling aspects of the Zcash miner's behavior and operational profile. Each configuration can be expressed via either the tool's command-line interface or within in an INI-sytle configuration file. The following details the facilities provided by this role to manage the contents of the aforementioned configuration file.

In typical `INI` or `TOML` fashion, individual sets of configurations consist of definitions representing config sections and associated setting key-value pairs. These sections and settings are made up of common miner options and custom Zcash server pool properties to access.

Reference [here](https://zec.nanopool.org/) for a list of server pool connection settings and [here](https://gist.github.com/0x0I/8a02072f7a2729bd8a0ab626d89c16d2) for an example config.

Each of these configurations can be expressed using the `zecminer_configs` hash, which contains a list of various Zecminer configuration options (hash) objects organized according to the following:
* common - miner configuration options common to all server connections
* server - custom server connection and operational configuration options to the specified server

Each `zecminer_configs` entry is a hash representing the equivalent of a configuration section as expected to be expressed by the `zecminer` server. As previously described, these hashes are structured by config section and associated key-value setting pairs. 

`[zecminer_configs: <entry>:] name: <common|server>` (**default**: *required*)
- name of the configuration section to render

`[zecminer_configs: <entry>:] settings: <YAML>` (**default**: )
- specifies parameters that manage various aspects of the Zcash miner's operations

###### Example

 ```yaml
  zecminer_configs:
    - name: common
      settings:
        log: 0
        logfile: /var/log/miner.log
        api: 127.0.0.1:42000
    - name: server
      settings:
        server: zec-us-east1.nanopool.org
        port: 6666
        user: user1-key
        pass: user1-password
  ```
  
##### Entrypoints

A part of what is termed as Traefik's "static configuration", entryPoints are the network entry points into Traefik. They define the port which will receive the requests (whether HTTP or TCP). Most of what happens to the connection between the clients and Traefik, and then between Traefik and the backend servers, is configured through the entrypoints in addition to routers.

See [here](https://docs.traefik.io/routing/entrypoints/) for more details regarding these options and suggestions on their usage.

`[traefik_configs: <entry>: config: entrypoints:] <YAML>` (**default**: )
- specifies parameters that manage Traefik entrypoint registration

###### Example

 ```yaml
  traefik_configs:
    - name: example-entrypoint
      # type: yaml
      # path: /etc/traefik
      config:
        entryPoints:
          web:
            address: ":80"
          websecure:
            address: ":443"
  ```
  
##### Routers

A router is in charge of connecting incoming requests to the services that can handle them. In the process, routers may use pieces of middleware to update the request, or act before forwarding the request to the service.

See [here](https://docs.traefik.io/routing/routers/) for more details regarding these options and suggestions on their usage.

`[traefik_configs: <entry>: config: <http|tcp>: routers] <YAML>` (**default**: )
- specifies parameters that manage Traefik router registration

###### Example

 ```yaml
  traefik_configs:
    - name: example-router
      config:
        http:
          routers:
            my-router:
              rule: "Path(`/foo`)"
              service: service-foo
  ```
  
##### Middlewares

Attached to the routers, pieces of middleware are a means of tweaking the requests before they are sent to your service (or before the answer from the services are sent to the clients). There are several available middleware in Traefik, some can modify the request, the headers, some are in charge of redirections, some add authentication, and so on.

Pieces of middleware can be combined in chains to fit every scenario.

See [here](https://docs.traefik.io/middlewares/overview/) for more details regarding these options and suggestions on their usage.

`[traefik_configs: <entry>: config: <http|tcp>: middlewares] <YAML>` (**default**: )
- specifies parameters that manage Traefik middleware registration

###### Example

 ```yaml
  traefik_configs:
    - name: example-middleware
      config:
        http:
          routers:
            router1:
              service: myService
              middlewares:
                - "foo-add-prefix"
              rule: "Host(`example.com`)"
          middlewares:
            foo-add-prefix:
              addPrefix:
                prefix: "/foo"
  ```
  
##### Services

Services are responsible for configuring how to reach the actual services that will eventually handle the incoming requests. Nested load balancers are able to load balance the requests between multiple instances of your services.

See [here](https://docs.traefik.io/routing/services/) for more details regarding available configuration settings and suggested usage.

`[traefik_configs: <entry>: config: <http|tcp>: services] <YAML>` (**default**: )
- specifies parameters that manage Traefik service registration

###### Example

 ```yaml
  traefik_configs:
    - name: example-service
      config:
        http:
          services:
            my-service:
              loadBalancer:
                servers:
                - url: "http://private-ip-server-1/"
                - url: "http://private-ip-server-2/"
  ```

##### Providers

Configuration discovery in Traefik is achieved through Providers. The providers are existing infrastructure components, whether orchestrators, container engines, cloud providers, or key-value stores. The idea is that Traefik will query the providers' API in order to find relevant information about routing, and each time Traefik detects a change, it dynamically updates the routes.

Traefik providers are categorized according to the following 4 groups:
* Label based (each deployed server/container has a set of labels attached to it)
* Key-Value based (each deployed server/container updates a key-value store with relevant information)
* Annotation based (a separate object, with annotations, defines the characteristics of the server/container)
* File based (the good old configuration file)

See [here](https://docs.traefik.io/providers/overview/#supported-providers) for a list of supported providers and more details regarding available configuration settings and suggested usage.

`[traefik_configs: <entry>: config: providers] <YAML>` (**default**: )
- specifies parameters that manage Traefik provider registration

###### Example

 ```yaml
  traefik_configs:
    - name: example-provider
      config:
        providers:
          consulCatalog:
            endpoint:
              address: http://127.0.0.1:8500
              # ...
  ```
  
#### Launch

This role supports launching a `traefik` server proxy utilizing the [systemd](https://www.freedesktop.org/wiki/Software/systemd/) service management tool, which manages the service as a background process or daemon subject to the configuration and execution potential provided by its underlying management framework.

_The following variables can be customized to manage the service's **systemd** [Service] unit definition and execution profile/policy:_

`extra_run_args: <traefik-cli-options>` (**default**: `[]`)
- list of `traefik` commandline arguments to pass to the binary at runtime for customizing launch.

Supporting full expression of `traefik`'s [cli](https://docs.traefik.io/reference/static-configuration/cli/) and, subsequently the full set of configuration options as referenced and described above, this variable enables the launch to be customized according to the user's exact specification.

`custom_unit_properties: <hash-of-systemd-service-settings>` (**default**: `[]`)
- hash of settings used to customize the `[Service]` unit configuration and execution environment of the *Traefik* **systemd** service.

#### Uninstall

Support for uninstalling and removing artifacts necessary for provisioning allows for users/operators to return a target host to its configured state prior to application of this role. This can be useful for recycling nodes and perhaps providing more graceful/managed transitions between tooling upgrades.

_The following variable(s) can be customized to manage this uninstall process:_

`perform_uninstall: <true | false>` (**default**: `false`)
- whether to uninstall and remove all artifacts and remnants of this `traefik` installation on a target host (**see**: `handlers/main.yml` for details)

Dependencies
------------

- 0x0i.systemd

Example Playbook
----------------
default example:
```
- hosts: all
  roles:
  - role: 0x0I.traefik
```

download and install specific version of Traefik binaries:
```
- hosts: all
  roles:
  - role: 0x0I.traefik
    vars:
      install_type: archive
      archive_url: https://github.com/containous/traefik/releases/download/v2.1.4/traefik_v2.1.4_linux_amd64.tar.gz
      archive_checksum: a0023d3b01c881cffc89b5b69d51d8e86d4e9ac87e87d57a9029731a83780893
```

enable debug logging for troubleshooting purposes:
```
- hosts: all
  roles:
  - role: 0x0I.traefik
    vars:
      traefik_configs:
        - name: log-settings
          config:
            log:
              level: DEBUG
              filePath: /var/log/traefik/traefik.log
              format: json
            accessLog:
              filePath: /var/log/traefik/access.log
              format: json
              bufferingSize: 100
              filters:
                statusCodes:
                - 500-511
                - 400-451
                - 308
              retryAttempts: true
              minDuration: 10ms
```

enable Prometheus metrics collection and reporting:
```
- hosts: all
  roles:
  - role: 0x0I.traefik
    vars:
      traefik_configs:
        - name: prometheus-metrics
          config:
            entryPoints:
              metrics:
                address: ":8082"
            metrics:
              prometheus:
                entryPoint: metrics
                addEntryPointsLabels: true
                addServicesLabels: true
                buckets:
                  - 0.1
                  - 0.3
                  - 1.2
                  - 5.0
```

configure Consul provider:
```
- hosts: all
  roles:
  - role: 0xOI.traefik
    vars:
      traefik_configs:
        - name: consul-provider
          config:
           providers:
             consulCatalog:
               stale: false
               cache: true
               exposedByDefault: false
               endpoint:
                 address: https://127.0.0.1:8500
                 scheme: https
                 datacenter: staging
               tls:
                 ca: path/to/ca.crt
                 caOptional: true
                 cert: path/to/foo.cert
                 key: path/to/foo.key
                 insecureSkipVerify: true
```

activate Jaeger tracing:
```
- hosts: all
  roles:
  - role: 0x0I.traefik
    vars:
      traefik_configs:
        - name: jaeger-tracing
          config:
            tracing:
              jaeger:
                samplingServerURL: http://localhost:5778/sampling
                samplingType: const
                samplingParam: 1.0
                localAgentHostPort: 127.0.0.1:6831
                gen128Bit: true
                traceContextHeaderName: example-trace-id
                endpoint: http://127.0.0.1:14268/api/traces?format=jaeger.thrift
```

License
-------

MIT

Author Information
------------------

This role was created in 2020 by O1.IO.
