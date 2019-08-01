## skuba: a multi-cluster MongoDB replica set deployment example

### introduction

This example outlines the steps for deploying a MongoDB replica set
with members in multiple clusters located in different public and
private cloud providers. The example addresses three-member replica
sets where each member of the replica set is deployed to its own
cluster. The application router network is used to enable member
communications for replica set formation and data transfer across the
multiple clusters.

### application router network

This example utilizes the application router network topology-1 as
detailed **here**. The topology includes clusters in two public cloud
providers and clusters in two private clouds. The application router network
provides connectivity across the separate clusters without the need
for special network and firewall configuration rules.

### cloud redundant replica set

To realize a cloud redundant three-member replica set deployment, one
member is deployed to one private cloud cluster and two members are 
deployed to the public cloud clusters. For the example, the member
in the private cloud will be designated as the primary for the replica
set and the members on the public cloud will serve as redundant
backups in case the member on the private cloud fails. In this way,
the remaining replica set members can provide data.

### replica set member deployment

TODO: create a project/namespace, same as topology deployment

This example provides three deployment yaml files, one for each member
of the replica set.

From the private cloud 1 console, setup the mongoDB member mongo-svc-a: 

```
oc apply -f deployment-mongo-svc-a.yaml
```

From the public cloud 1 console, setup the mongoDB member mongo-svc-b: 

```
oc apply -f deployment-mongo-svc-b.yaml
```

From the public cloud 2 console, setup the mongoDB member mongo-svc-c: 

```
oc apply -f deployment-mongo-svc-c.yaml
```

### replica set formation

From the private cloud 1 console, use the mongo shell to connect to
the mongo-svc-a instance and initiate the member set formation:

```
mongo --host $(oc get service mongo-svc-a -o=jsonpath='{.spec.clusterIP}')
>load("replica.js")
```

The replica set should elect a primary. You can view the status of
the members array in the mongo shell using:

```
>rs.status()
```

### observing replication

While staying connected to the mongo-svc-a shell, insert some documents:

```
> use test
> for (i=0; i<1000; i++) {db.coll.insert({count: i})}
>
> // make sure the docs are there
> db.coll.count()
```

Next, check one of the secondaries and verify they they have a copy of
all the documents. Acquire a connection to one of the secondaries by
instantiating a connection object using the `Mongo()` constructor
within the currently active shell:

TODO: either add addresses to /etc/hosts or lookup address for svc

```
> msbConn = new Mongo("mongo-svc-b")
> msbDB = msbConn.getDB("test")
> msbDB.coll.find()
> msbDB.setSlaveOk() 
```
And do the same for the other secondary:

```
> mscConn = new Mongo("mongo-svc-c")
> mscDB = mscConn.getDB("test")
> mscDB.coll.find()
```

TODO: example of primary failover, how to initiate and observe
