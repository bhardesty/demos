## Simple HTTP and TCP services across multiple clusters

This demonstration shows HTTP and TCP service access across a network of connected clusters.

> Note: This demonstration currently runs only on OpenShift (it uses routes and some
> other OpenShift-specific APIs).
>
> TODO: Update this demo to support both OpenShift and Kubernetes

### Set up a network of clusters

Use the _skoot_ project to generate the yaml configurations for each cluster.

An example 4-cluster topology is provided in `network-setup.example`.  Make a copy of this
file called `network-setup`, replace `$ROUTING_SUFFIX` for pub1 and pub2 with the actual
suffixes for your clusters.

Note that it is valid to use a different network setup if you have fewer (or more)
available clusters you would like to use.  Any Router/Cluster that is going to be
connected to (appears in the second argument of a Connect line), must be provided
a route hostname (inter-router.demo.$ROUTING_SUFFIX) and must be reachable via TCP/IP
from the connecting clusters.

Run the skoot tool on `network-setup` and get the resulting four yaml files.

On each cluster, apply the respective yaml file using:

```
oc apply -f $CONFIG.yaml
```

### Checking the status of the network

In the example network-setup, a `Console` route is specified for the laptop cluster.
This route may be followed to launch the Apache Qpid Dispatch Router console.  In the console,
the _topology_ tab can be selected to show the routers in the network.  If everything is
working as expected, you should see all of your routers in the topology.

### Deploy the HTTP and TCP services on the `pvt1` cluster

Log in to the pvt1 cluster at the command line.

Set up the service-account and router-access secret for the proxies:

```
oc apply -f proxy-setup.yaml
```

Launch the HTTP services:

```
oc apply http/deployment.yaml
```

Launch the TCP services:

```
oc apply -f tcp/deployment.yaml
```

Launch the HTTP proxy:

```
oc apply -f http/service-proxy.yaml
```

Launch the TCP proxy:

```
oc apply -f tcp/service-proxy.yaml
```

### Deploy only the proxies on the `laptop` cluster

Log into the laptop cluster at the command line.

Set up the service-account and router-access secret for the proxies:

```
oc apply -f proxy-setup.yaml
```

Launch the HTTP proxy:

```
oc apply -f http/service-proxy.yaml
```

Launch the TCP proxy:

```
oc apply -f tcp/service-proxy.yaml
```

### Test availability of the remote services

To access the HTTP service from `laptop`:

```
curl $(oc get service myservice -o=jsonpath='{.spec.clusterIP}'):8080/foo -H host:myservice
```

To access the TCP services from `laptop`:

```
telnet $(oc get service echo -o=jsonpath='{.spec.clusterIP}') 9090
```

### Variations on the demo

You can add the services and proxies to other clusters to demonstrate
load sharing.  Note that in order to cause load to be shared across
different clusters, you will need to create a substantial amount of
concurrent load.


