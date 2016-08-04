# Quick start using docker

[Docker](docker.io) is a toolkit and a daemon which makes running foreign applications convenient, via containers.
This tutorial will help you run metrictank, its dependencies, and grafana for dashboarding, with minimal hassle.

[docker installation instructions](https://www.docker.com/products/overview)
You will also need to install [docker-compose](https://docs.docker.com/compose/)

First go into the `docker` dir of this project.
You can bring up the stack like so:

```
docker-compose up
```

A bunch of text will whiz past on your screen, but you should see

```
metrictank_1     | waiting for cassandra:9042 to become up...
statsdaemon_1    | 2016/08/04 12:31:21 ERROR: dialing metrictank:2003 failed - dial tcp 172.18.0.5:2003: getsockopt: connection refused
metrictank_1     | waiting for cassandra:9042 to become up...
statsdaemon_1    | 2016/08/04 12:31:22 ERROR: dialing metrictank:2003 failed - dial tcp 172.18.0.5:2003: getsockopt: connection refused
```

And a little bit later:

```
metrictank_1       | 2016/08/04 11:28:24 [I] DefCache initialized in 50.40667ms. starting data consumption
metrictank_1       | 2016/08/04 11:28:24 [I] carbon-in: listening on :2003/tcp
metrictank_1       | 2016/08/04 11:28:24 [I] starting listener for metrics and http/debug on :6060
```

Once the stack is up, metrictank should be running on port 6060:

```
$ curl http://localhost:6060
OK
$ curl http://localhost:6060/cluster
{"instance":"default","primary":true,"lastChange":"2016-08-02T17:12:25.339785926Z"}
```

Then, in your browser, open Grafana which is at `http://localhost:3000` and log in as `admin:admin`
In the menu upper left, hit `Data Sources` and then the `add data source` button.
Add a new data source with name `metrictank`, check "default", type `Graphite`, uri `http://localhost:8080` and access mode `direct` (not `proxy`).

When you hit save, Grafana should succeed in talking to the data source.

![Add data source screenshot](https://raw.githubusercontent.com/raintank/metrictank/master/docs/img/add-datasource-docker.png)

Now let's see some data.  If you go to `Dashboards`, `New` and add a new graph panel.
In the metrics tab you should see a bunch of data already: 

* data under `stats`: these are metrics coming from metrictank and graphite-api.  
  i.e. they send their own instrumentation into statsd (statsdaemon actually is the version we use here),  
  and statsdaemon sends aggregated metrics into metrictank's carbon port.  Statsdaemon flushes every second.
* statsdaemon's own internal metrics which it sends to metrictank's carbon port.
* after about 5 minutes you'll also have some usage metrics show up under `metrictank`. See usage.md

Note that metrictank is setup to track every metric on a 1-second granularity.  If you wish to use it for less frequent metrics,
you have to modify the storage-schemas.conf, just like with graphite.

You can also send your own data into metrictank using the carbon input, like so:

```
echo "example.metric 123 $(date +%s)" | nc localhost 2003
```

TODO: import MT own dashboard


Finally, you can tear down the entire stack like so:
```
docker-compose stop
```

To clean up all data so you can start fresh, run this after you stopped the stack:
```
docker rm -v $(docker ps -a -q -f status=exited)
```
This will remove the stopped containers and their data volumes.