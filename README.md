# FaST infra setup

The FaST platform makes use of several technologies such as MongoDB, Kafka or Nuclio (serverless framework) that are deployed alongside the core services. The declarative configuration of these bricks are centralized in this repository.

The Proof of Concept of the FaST platform has been deployed on the [Sobloo](https://sobloo.eu) DIAS.

## Core services

Please refer directly to the following repositories for the deployment of the core services:

**Data ingestion pipelines**:
* [hydrology es](https://github.com/PwC-FaST/fast-core-hydrology-es)
* [hydrology fr](https://github.com/PwC-FaST/fast-core-hydrology-fr)
* [lpis es](https://github.com/PwC-FaST/fast-core-lpis-es)
* [lpis fr](https://github.com/PwC-FaST/fast-core-lpis-fr)
* [natura2000](https://github.com/PwC-FaST/fast-core-natura2000)
* [soc](https://github.com/PwC-FaST/fast-core-soil-soc)
* [topsoil](https://github.com/PwC-FaST/fast-core-soil-topsoil)

**GIS API**:
* [parcel-info](https://github.com/PwC-FaST/fast-core-gis-parcelinfo)
* [parcel-svg](https://github.com/PwC-FaST/fast-core-gis-parcelsvg)

**FaST** [tile server](https://github.com/PwC-FaST/fast-core-gis-tileserver)

## FaST webapp

The FaST webapp makes use of several secret keys which need to be set in the ```fast-webapp/webapp/secret.yaml```file:
* django-secret-key
* postgres-db-user
* postgres-db-password
* dark-sky-api-key
* mapquest-api-key
* fast-webapp-default-django-password

Please refer to the [FaST project](https://github.com/PwC-FaST/fast-webapp) for more information.

A basic configuration is provided and works as is. Don't forget to update the configuration according to your environment (ex: the number of replicas, docker images and ingress settings).

To deploy the webapp and a dedicated Postgres database, apply the following YAML manifests:

```bash
$> cd fast-webapp
$> kubectl create -f postgres
$> kubectl create -f webapp
```

Then check the status of the Kubernetes pods:

```bash
$> kubectl -n fast-platform get pod

NAME                                                           READY     STATUS    RESTARTS   AGE
fast-webapp-76b8bcff98-hwcwb                                   2/2       Running   0          1d
fast-webapp-postgres-6c5d7957f4-7hjbb                          1/1       Running   0          1d
...
```

## Kafka

To deploy a Kafka cluster of three brokers, apply the following YAML manifests:

```bash
$> cd kafka/

$> kubectl create -f 00-namespace.yaml
$> kubectl create -f zookeeper/
$> kubectl create -f kafka/
```

Then check the status of the Kubernetes pods:

```bash
$> kubectl -n kafka get pod

NAME                        READY     STATUS    RESTARTS   AGE
kafka-0                     2/2       Running   0          1d
kafka-1                     2/2       Running   0          1d
kafka-2                     2/2       Running   0          1d
zookeeper-0                 1/1       Running   0          1d
zookeeper-1                 1/1       Running   0          1d
zookeeper-2                 1/1       Running   0          1d
```

To test the Kafka cluster, deploy the kafka-test container and check the logs on the standard output:

```bash
$> kubectl create -f client-test.yaml
$> kubectl -n kafka logs kafka-test

Created topic "helm-test-topic-create-consume-produce".
Processed a total of 1 messages
>>Tue Jan 15 14:07:17 UTC 2019
```

## MongoDB

To deploy a MongoDB replicaset (1 primary node, 2 secondary nodes and 1 arbiter node), apply the following YAML manifests:

```bash
$> cd mongodb/

$> kubectl create -f 00-namespace.yaml
$> kubectl create -f replicaset
```

Then check the status of the Kubernetes pods:

```bash
$> kubectl -n mongodb get pod

NAME                  READY     STATUS    RESTARTS   AGE
mongodb-arbiter-0     1/1       Running   0          1d
mongodb-primary-0     1/1       Running   0          1d
mongodb-secondary-0   1/1       Running   0          1d
mongodb-secondary-1   1/1       Running   0          1d
```

To connect to the MongoDB replicaset and start a mongo shell:

```bash
$> kubectl -n mongodb exec -ti mongodb-primary-0 bash
$> mongo
rs0:PRIMARY> show dbs

admin    0.000GB
config   0.000GB
fast    21.304GB
local    0.986GB
```

## Nuclio

In the ```nuclio/0.5.3/registry.yaml``` file, set the key of your own Docker registry.

To deploy Nuclio and its ```function``` CRD (Custom Resource Definition), apply the following YAML manifests:

```bash
$> cd nuclio

$> kubectl create -f 00-namespace.yaml
$> kubectl create -f 0.5.3
```

Then check the status of the Kubernetes pods:

```bash
$> kubectl -n nuclio get pod

NAME                                READY     STATUS    RESTARTS   AGE
nuclio-controller-fccf87cb9-hlk25   1/1       Running   0          1d
```

Please refer to the [official Nuclio documentation](https://nuclio.io/docs/latest/) for more information.
