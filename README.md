Watcher Installation
====================

To test Watcher, you can deploy it algonside DevStack and run Watcher in a
Docker environment. Watcher image containers integrate automatic
provisioning scripts to simplify installation and configuration. You can
install DevStack and Docker tools on the same machine.

You have to be familiar with Docker and its [Docker Compose] orchestration
tool.
  [Docker Compose]: https://docs.docker.com/compose/

Install devstack
----------------

If you need to install DevStack, you can refer to this page : [DevStack - an OpenStack Community Production]. We recommend using a minimal install of Ubuntu or Fedora server in a new VM.
``` sh
  $ git clone https://git.openstack.org/openstack-dev/devstack
  $ cd devstack
  $ git checkout stable/kilo 
  $ ./stack.sh
```
Note: If you deploy DevStack on a NATed virtual machine, don't forget to udpate the DevStack default configuration (in the file *local.conf*) by setting the *HOST_IP* parameter with the IP address of your physical machine. for more information, go to [DevStack Configuration].

Install Docker & [Docker Compose]
---------------------------------

To install [Docker], you can refer to this page : [Supported installation]
and choose your distribution.

1.  On Ubuntu simply do:
    ``` sh
      $ sudo apt-get install curl
      $ curl -sSL https://get.docker.com | sh
    ```
2.  To test your installation:
    ``` sh
      $ sudo docker run hello-world
    ```

To install Docker compose, you can refer to this page: [Install Docker Compose]:
``` sh
$ curl -L https://github.com/docker/compose/releases/download/1.4.0/docker-compose-)\`uname -s\`-\`uname -m\` \> /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```
  [DevStack - an OpenStack Community Production]: http://docs.openstack.org/developer/devstack/
  [DevStack Configuration]: http://docs.openstack.org/developer/devstack/configuration.html
  [Supported installation]: https://docs.docker.com/installation/
  [Install Docker Compose]: https://docs.docker.com/compose/install/
  [Docker]: https://docs.docker.com/


Watcher containers
==================

Containers orchestration templating is composed of 3 groups of containers, with following roles:

-   *__SERVICE\_DISCOVERY__* : containers providing an automatic service discovery tool, based on:
     -   [consul],
     -   [registrator]

-   *__WATCHER__* : containers providing a basic OpenStack identity Service and the Watcher modules (including Horizon plugin):
     -   [mariadb],
     -   [rabbitmq],
     -   [keystone],
     -   console,
     -   api,
     -   decision-engine,
     -   applier,
     -   horizon

-  *__METERING__*: containers providing a complete metering chain for Watcher:
     -   [riemann],
     -   [influxdb],
     -   nanoconfig-server,
     -   metering-publisher,
     -   metering-agent,
     -   [grafana]

  [consul]: https://github.com/hashicorp/consul
  [registrator]: https://github.com/gliderlabs/registrator
  [mariadb]: https://mariadb.org/
  [rabbitmq]: https://www.rabbitmq.com/
  [riemann]: http://riemann.io/
  [influxdb]: https://influxdb.com/
  [grafana]: http://grafana.org/
  [keystone]: http://docs.openstack.org/developer/keystone/

Run the Service Discovery tool
==============================
  Please see [service_discovery/README.md](service_discovery/README.md) 
  
Run the Watcher Service
=======================
  Please see [watcher/README.md](watcher/README.md) 

Run the Watcher Metering Chain
==============================
  Please see [metering/README.md](metering/README.md) 
