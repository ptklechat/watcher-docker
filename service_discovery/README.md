Install Service Discovery tools
===============================

-   Service discovery containers are:
     -   [consul],
     -   consuldata,
     -   [registrator]

To start the *consul* server in bootstrapped mode:
``` sh
    $ cd service-discovery
    $ docker-compose up -d
```
Note that *consul* exposes the default DNS port (53) to the *docker0* bridge interface (172.17.42.1), this interface will be added to the *resolv.conf* file for all containers using the *-dns* command.

Registrator is configured with the *-internal* in order to assign to services registered by containers their docker-internal ip instead of the host one.

[consul]: https://github.com/hashicorp/consul
[registrator]: https://github.com/gliderlabs/registrator