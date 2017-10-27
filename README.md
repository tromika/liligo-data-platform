# liligo Data Platform

Initially we wanted a platform where we can run heterogeneous workload distributed across our on-premise infra. We started to use Apache Spark as a unified computing engine but we needed a lightweight environment where we can run Spark applications. Furthermore, our Devops team introduced Docker as the core technology on our servers so we wanted to align with their direction.

Mesos on Docker was a great start it's really flexible and reproducible. We're really satisfied Spark on Mesos as well. This stack supports our batch jobs, ad-hoc queries, and all data science related workloads we have.

For more information please visit http://docker.com , https://mesosphere.com/mesos and https://zookeeper.apache.org/

## Getting started

* Install [Docker](https://www.docker.com/)
* You can get our pre-built Docker images from https://hub.docker.com/r/lilidata/
* Customise the docker-compose config and you can launch the platform

## Components

* [Exhibitor](https://hub.docker.com/r/lilidata/exhibitor) - Zookeeper for Mesos
* [Mesos](https://hub.docker.com/r/lilidata/mesos) - Mesos Master and Agent
* [Kafka-Mesos](https://hub.docker.com/r/lilidata/kafka-mesos/) - Kafka on Mesos
* [HDFS-Mesos](https://hub.docker.com/r/lilidata/hdfs-mesos/) - HDFS on Mesos

### Exhibitor

It's a Zookeeper basically. Typically we launched Exhibitor nodes for each Mesos Master node so we run both on a Docker host. In our setup we launch Master and Exhibitor pairs to fix hosts.

With the [shared/network configuration](https://github.com/soabase/exhibitor/wiki/Shared-Configuration) option you can share the configuration file with all zookeeper nodes. We persist the configuration file to local disk and NFS as well at the same time. Also we use it with [automatic instance management](https://github.com/soabase/exhibitor/wiki/Automatic-Instance-Management) so new nodes get the configuration via NFS and they can add themselves to the list.

### Mesos

For the Mesos Master and Agent you have to specify only the exhibitor nodes and the path for your Mesos cluster.

The base Mesos image can be used to spawn Master or Agent nodes so only the entrypoint and some parameters different. Added/Removed Agent nodes are automatically registered at the leading Mesos Master.

If you share your host machine's docker pid with your Mesos Agent container you can leverage the host's docker capabilities so basically you can launch separate containers as executors from your Agent containers.

The current setup is useful to spawn Apache Spark executors quickly so that's why the main Spark versions are already in the Docker image. You can point Spark to the right directory on executors/agents with spark.mesos.executor.home . For more info please visit https://spark.apache.org/docs/latest/running-on-mesos.html


## Configuration

Please check the docker-compose.yml for the configuration but We describe some Mesos related parameters here.

* MESOS_GC_DELAY - Spark can create a lot of log files under Mesos so you can specify the retention for finished frameworks.
* MESOS_CONTAINERIZERS=docker,mesos - You can launch Mesos executors and Docker containers as executors. http://mesos.apache.org/documentation/latest/mesos-containerizer/
* pid:"host" and mount of /var/run/docker.sock:/var/run/docker.sock is needed to be able to access host's Docker process
* Could be useful to mount host directory and use it for Mesos frameworks as a working directory or for logs so you can persist data on the host. https://docs.docker.com/engine/admin/volumes/volumes/
* We recommend to use host networking
* MESOS_QUORUM - Please visit http://mesos.apache.org/documentation/latest/operational-guide/ for determine your quorum size.

## Frameworks

We use the following frameworks on Mesos internally:

* Apache Spark - For batch processing, Streaming microservices and interactive workload with [Apache Zeppelin](https://zeppelin.apache.org/)  - https://spark.apache.org/docs/latest/running-on-mesos.html
* HDFS - As our datalake - https://github.com/elodina/hdfs-mesos
* Kafka - For real-time data - https://github.com/mesos/kafka
* Marathon - For long running services and frameworks - https://mesosphere.github.io/marathon/

Please visit http://mesos.apache.org/documentation/latest/frameworks/ for the complete list of frameworks that supported on Mesos.

## License

Licensed under the Apache License, Version 2.0: http://www.apache.org/licenses/LICENSE-2.0
