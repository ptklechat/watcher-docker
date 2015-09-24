Watcher Metering Chain Installation
===================================

Architecture
------------
Please have a look on the [Watcher metering architecture] before to deploy the complete chain.
[Watcher metering architecture]: https://factory.b-com.com/www/watcher/watcher-metering

Watcher Metering containers
===========================

Watcher metering containers are:
 -   metering-agent
 -   metering-publisher
 -   riemann
 -   influxdb
 -   nanoconfig-server
 -   grafana

*Watcher-metering-agent* container is only used for development. This agent must be deployed on compute nodes or on a DevStack platform. Please refer to the [Manual Installation of Watcher Metering Agent] section for more information. 

If you don't need to deploy a metering agent container, edit the orchestration file and remove/comment the metering-agent service:
```sh
  $ cd metering
  $ vi docker-compose.xml
```

Customize the orchestration template file
=========================================

If you install a metering agent on a compute node (not included into docker runtime environment), you have to customize the file *docker-compose.xml* file to allow this metering agent to send metrics to the metering publisher container.

Data push from agents to publishers is performed by [nanomsg](http://nanomsg.org/index.html), but can be achieved in two distinct ways:
- using pipeline [One-Way pipe](http://tim.dysinger.net/posts/2013-09-16-getting-started-with-nanomsg.html) between *agent* (NN_PUSH) and *publisher* (NN_PULL). With this mode *nanoconfig-server* service is useless.
- using *nanomsg* service reconfiguration: [nanoconfig](https://github.com/nanomsg/nanoconfig). With this mode, both *agent* and *publisher* request to the *nanoconfig-server* the topology to apply.

By default, the *nanoconfig* way is used : this is set by the environment variable `USE_NANOCONFIG_SERVICE` (*true* or *false*).

In production target, many publishers can be instantiated and load-balanced. For simplicity, we will only instantiate one publisher service in this example.

#### Customize for direct connection between agents and publisher
Because the *agent* is installed outside the docker runtime environment, the *publisher* port must be exposed to the host with the *-ports "HOST:CONTAINER"* command. 
Following steps must be applied to the *metering-publisher* service:
1. Set `USE_NANOCONFIG_SERVICE` to false (default is *true*)
2. Expose the listening port 12345 (binded on the host port 54321):
``` yaml
  ports:
     - "54321:12345"
```
3. Set `PUBLISHER_ENDPOINT` to *tcp://0.0.0.0:12345*
4. Comment  `NANOCONFIG_*` variables .

The *nanoconfig-server* service is useless and should be commented.

#### Customize for nanoconfig service (nanomsg reconfiguration)
Following steps must be applied to the *nanoconfig-server* service:
1. Expose the listening ports for *nanoconfig* setup (10800) and update (10900):
``` yaml
  ports:
    - "10800:10800"
    - "10900:10900"
```
2. `NANOCONFIG_AGENT_PROFILE` and `NANOCONFIG_PUBLISHER_PROFILE` can be let to their default values
2. Set the `WATCHER_METERING_PUBLISHER_ENDPOINT` url
``` yaml
WATCHER_METERING_PUBLISHER_ENDPOINT: tcp://<HOST_IP>:54321
```
where `HOST_IP` is the external IP address of the metering publisher container (i.e. IP address of the physical machine hosting the Docker Engine)

Following steps must be applied to the *metering-publisher* service:
1. Set `USE_NANOCONFIG_SERVICE` to true
2. `NANOCONFIG_PUBLISHER_PROFILE` can be let to its default value
3. Expose the listening port 12345 (binded on the host port 54321):
``` yaml
  ports:
     - "54321:12345"
```
3. Set `NANOCONFIG_SERVICE_ENDPOINT` to *tcp://<HOST_IP>:10800* and `NANOCONFIG_UPDATE_ENDPOINT` to *tcp://<HOST_IP>:10900* where `HOST_IP` is the external IP address (same as above)
4. `PUBLISHER_ENDPOINT` is useless.

Run the Metering Chain
======================
Deploy the containers:

``` sh
    $ cd metering
    $ docker-compose up -d
```

Manual Installation of Watcher Metering Agent
=============================================

System prerequisites
--------------------

As this project is targeting an OpenStack environment, please refer to their [installation procedure] to find out more about the required packages
[installation procedure]: http://docs.openstack.org/developer/nova/devref/development.environment.html#explicit-install-clone
As of now, their Ubuntu requirements are:
```sh
  $ sudo apt-get install python-dev libssl-dev python-pip git-core libxml2-dev libxslt-dev pkg-config libffi-dev libpq-dev libmysqlclient-dev libvirt-dev graphviz libsqlite3-dev python-tox
```

Install packages and dependencies
---------------------------------

The watcher metering agent is part of a package called *python-watcher_metering* including both the metering publisher and the metering agent. Moreover, this project only provides the core logic and hence needs metric drivers. These drivers can be found in a separate package called *python-watcher_metering_drivers*

The watcher metering agent and the metric drivers can be installed using *pip* and  PyPI (i.e. Python Package Index). To do so, this can be installed as followed:
```sh
  $ pip install python-watcher_metering
  $ pip install python-watcher_metering_drivers
```

This package can be installed either globally or within a *virtualenv*. Considering *VENV_ROOT_DIR* as the root folder of you virtual environment, you can install it as followed: 
```sh
  $ virtualenv $(VENV_ROOT_DIR)
  $ source $(VENV_ROOT_DIR)/bin/activate
  $ pip install python-watcher_metering
  $ pip install python-watcher_metering_drivers
```

Configuration
-------------

Both the watcher metering agent and the metrics drivers can be configured via configuration files. A sample is shipped with each of these packages:

- An *agent.conf* file for the *python-watcher_metering* package.
- A *watcher-metering-drivers.conf* file for the *python-watcher_metering_drivers* package.

Both of these files are installed based on the type of install you chose:

- If you installed it globally (without using a *virtualenv*), these configuration files can be found within the `/usr/local/etc/watcher-metering` folder.
- If you installed it within a virtualenv, these configuration files can be found within the `$(VENV_ROOT_DIR)/etc/watcher-metering` folder, where *VENV_ROOT_DIR* is the root folder of you virtual environment.
