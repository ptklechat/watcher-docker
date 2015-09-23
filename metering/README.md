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


*Watcher-metering-agent* container is only used for development. Agent agent must be deployed on compute nodes or on a DevStack platform. Please see section [Manual Installation of Watcher Metering Agent]. 

If you don't need to deploy a metering agent container, edit the orchestration file and remove/comment the metering-agent service:
```sh
$ cd metering
$ vi docker-compose.xml
```

Customize the orchestration template file
=========================================

If you install a metering agent on a compute node (not included into docker runtime environment), you have to customize the file *docker-compose.xml* file to allow this metering agent to send metrics to the metering publisher container.

Data push from agents to publishers is performed by [nanomsg](http://nanomsg.org/index.html), but can be achieved two distincts ways:
- using pipeline [One-Way pipe](http://tim.dysinger.net/posts/2013-09-16-getting-started-with-nanomsg.html) between *agent* (NN_PUSH) and *publisher* (NN_PULL). With this mode *nanoconfig-server* service is useless.
- using nanomsg service reconfiguration: [nanoconfig](https://github.com/nanomsg/nanoconfig). With this mode, both *agent* and *publisher* request to the *nanoconfig-server* the topology to apply.

By default, the *nanoconfig* way is used : this is set by the environment variable `USE_NANOCONFIG_SERVICE` (*true* or *false*).

In production target, many publishers can be instanciated and load-balanced. For simplicity, we will only instanciate one publisher service in this example.

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

    $ cd metering
    $ docker-compose up -d


Manual Installation of Watcher Metering Agent
=============================================

Install packages and dependencies
---------------------------------

Configuration
-------------
