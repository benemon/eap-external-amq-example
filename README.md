# MDB Example with External Artemis Broker

## Introduction

This repository demonstrates how to configure Red Hat JBoss EAP 7.1 to use an external Red Had JBoss AMQ7 as its Message Broker. It is not meant as an exhaustive tutorial on the use of either product.

The setup steps for this project were sourced from the [official documentation for Red Hat JBoss EAP](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html/configuring_messaging/resource_adapters#using_jboss_amq_for_remote_jms_communication). Please refer directly to this for further information.

## Prerequisites

* [Red Hat JBoss EAP 7.1](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=appplatform&downloadType=distributions})

* [Red Hat JBoss AMQ 7](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=jboss.amq.broker)

* Git clone of this repo!

Download both products and extract them into a convenient directory alongside the clone of this repository. For the sake of the exercise, it is best to keep them in the same top level directory. Once both archives have been extracted, you should have a top level folder with three sub folders:

```
$ ll
total 1558784
drwxr-xr-x   19 eapuser  staff        608 27 Feb 09:11 .
drwx------+  51 eapuser  staff       1632 19 Feb 21:17 ..
drwxr-xr-x@  24 eapuser  staff        384 27 Feb 06:39 eap-external-amq-example
drwxr-xr-x@  12 eapuser  staff        384 27 Feb 06:36 amq-broker-7.0.3
drwxrwxr-x@  17 eapuser  staff        544 27 Feb 07:00 jboss-eap-7.1
```

## Setup AMQ7

* Navigate to the amq distribution's`bin` directory and create the broker.

```
$ ./artemis create --user eapUser --password eapUser eapBroker ../../eapBroker --require-login
Creating ActiveMQ Artemis instance at: /Users/eapuser/Documents/product/eapBroker

Auto tuning journal ...
done! Your system can make 16.67 writes per millisecond, your journal-buffer-timeout will be 59999

You can now start the broker by executing:  

   "/Users/eapuser/Documents/product/eapBroker/bin/artemis" run

Or you can run the broker in the background using:

   "/Users/eapuser/Documents/product/eapBroker/bin/artemis-service" start
```

* You should now have four directories in your top level directory from the prerequisites:

```
$ ll
total 1558887
drwxr-xr-x   19 eapuser  staff        608 27 Feb 09:11 .
drwx------+  51 eapuser  staff       1632 19 Feb 21:17 ..
drwxr-xr-x@  24 eapuser  staff        384 27 Feb 06:39 eap-external-amq-example
drwxr-xr-x@  12 eapuser  staff        384 27 Feb 06:36 amq-broker-7.0.3
drwxrwxr-x@  17 eapuser  staff        544 27 Feb 07:00 jboss-eap-7.1
drwxr-xr-x    7 eapuser  staff        224 27 Feb 09:26 eapBroker
```

* Navigate to the new broker's `etc` directory - `eapBroker/etc`. Message destinations have to be configured explicitly when being used with EAP (rather than utilising standard AMQ behaviour of autocreating destinations). These are added to the file `etc/broker.xml`:

 ```
<configuration xmlns="urn:activemq"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="urn:activemq /schema/artemis-configuration.xsd">

    <core xmlns="urn:activemq:core" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="urn:activemq:core ">
        ....
        <addresses>
            <address name="HELLOWORLDMDBQueue">
                <anycast>
                    <queue name="HELLOWORLDMDBQueue" />
                </anycast>
            </address>
        <addresses>
        ....
    </core>
</configuration>
 ```
 **NOTE: Sections omitted for brevity**
 
 * Start the broker from the broker's `bin` directory - `eapBroker/bin`. You should see output similar to the following:
 
 ```
 $ ./artemis run
 SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
 SLF4J: Defaulting to no-operation (NOP) logger implementation
 SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
            __  __  ____    ____            _
      /\   |  \/  |/ __ \  |  _ \          | |
     /  \  | \  / | |  | | | |_) |_ __ ___ | | _____ _ __
    / /\ \ | |\/| | |  | | |  _ <| '__/ _ \| |/ / _ \ '__|
   / ____ \| |  | | |__| | | |_) | | | (_) |   <  __/ |
  /_/    \_\_|  |_|\___\_\ |____/|_|  \___/|_|\_\___|_|
 
  Red Hat JBoss AMQ 7.0.3.GA
 
 
 09:34:27,410 INFO  [org.apache.activemq.artemis.integration.bootstrap] AMQ101000: Starting ActiveMQ Artemis Server
 09:34:27,484 INFO  [org.apache.activemq.artemis.core.server] AMQ221000: live Message Broker is starting with configuration Broker Configuration (clustered=false,journalDirectory=./data/journal,bindingsDirectory=./data/bindings,largeMessagesDirectory=./data/large-messages,pagingDirectory=./data/paging)
 09:34:27,509 INFO  [org.apache.activemq.artemis.core.server] AMQ221013: Using NIO Journal
 09:34:27,566 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-server]. Adding protocol support for: CORE
 09:34:27,567 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-amqp-protocol]. Adding protocol support for: AMQP
 09:34:27,567 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-hornetq-protocol]. Adding protocol support for: HORNETQ
 09:34:27,567 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-mqtt-protocol]. Adding protocol support for: MQTT
 09:34:27,568 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-openwire-protocol]. Adding protocol support for: OPENWIRE
 09:34:27,568 INFO  [org.apache.activemq.artemis.core.server] AMQ221043: Protocol module found: [artemis-stomp-protocol]. Adding protocol support for: STOMP
 09:34:27,617 INFO  [org.apache.activemq.artemis.core.server] AMQ221034: Waiting indefinitely to obtain live lock
 09:34:27,618 INFO  [org.apache.activemq.artemis.core.server] AMQ221035: Live Server Obtained live lock
 09:34:27,698 INFO  [org.apache.activemq.artemis.core.server] AMQ221003: Deploying queue HELLOWORLDMDBQueue
 09:34:28,060 INFO  [org.apache.activemq.artemis.core.server] AMQ221020: Started NIO Acceptor at 0.0.0.0:61616 for protocols [CORE,MQTT,AMQP,STOMP,HORNETQ,OPENWIRE]
 09:34:28,063 INFO  [org.apache.activemq.artemis.core.server] AMQ221020: Started NIO Acceptor at 0.0.0.0:5445 for protocols [HORNETQ,STOMP]
 09:34:28,065 INFO  [org.apache.activemq.artemis.core.server] AMQ221020: Started NIO Acceptor at 0.0.0.0:5672 for protocols [AMQP]
 09:34:28,067 INFO  [org.apache.activemq.artemis.core.server] AMQ221020: Started NIO Acceptor at 0.0.0.0:1883 for protocols [MQTT]
 09:34:28,070 INFO  [org.apache.activemq.artemis.core.server] AMQ221020: Started NIO Acceptor at 0.0.0.0:61613 for protocols [STOMP]
 09:34:28,071 INFO  [org.apache.activemq.artemis.core.server] AMQ221007: Server is now live
 09:34:28,071 INFO  [org.apache.activemq.artemis.core.server] AMQ221001: Apache ActiveMQ Artemis Message Broker version 2.0.0.amq-700013-redhat-1 [0.0.0.0, nodeID=677d4e45-1ba1-11e8-b7b8-4c32759789f7] 
 INFO  | main | Initialized dispatch-hawtio-console plugin
 09:34:30,629 INFO  [org.apache.activemq.artemis] AMQ241001: HTTP Server started at http://localhost:8161
 09:34:30,629 INFO  [org.apache.activemq.artemis] AMQ241002: Artemis Jolokia REST API available at http://localhost:8161/jolokia
 ```
 
## Setup EAP7.1

* Navigate to EAP's `bin` folder, and execute the following JBoss CLI script. This script automates the steps outlined in the official documentation to set up the `standalone-full.xml` profile for use with an external AMQ7 broker.

`$ ./jboss-cli.sh --file=../../eap-external-amq-example/configuration/eap-artemis.cli `

* The final piece of configuration of the EAP instance is to set the default Connection Factory for all MDBs to use, as by default they'll just use the internal Connection Factory. This can be done by setting the `ejb.resource-adapter-name` property at startup. This can be defined using any of the standard property configuration mechanisms, but in this example a command line property is used.

`-Dejb.resource-adapter-name=activemq-ra-remote`

* With this in mind, start the EAP server instance with the `standalone-full.xml` profile to validate successful configuration.

`$ ./standalone.sh -c standalone-full.xml -Dejb.resource-adapter-name=activemq-ra-remote`

* After a few seconds you should see a message similar to the following:

```
17:00:21,092 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0060: Http management interface listening on http://127.0.0.1:9990/management
17:00:21,093 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0051: Admin console listening on http://127.0.0.1:9990
17:00:21,093 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: JBoss EAP 7.1.0.GA (WildFly Core 3.0.10.Final-redhat-1) started in 6044ms - Started 490 of 716 services (372 services are lazy, passive or on-demand)
```

## Build Demo Project

* From the top level directory structure, navigate to the git cloned project containing the Maven `pom.xml` and run the build:

`$ cd eap-external-amq-example`

`$ mvn clean install`

* Copy the resulting WAR file from `target/helloworld-mdb.war` to your EAP distribution, under `standalone/deployments`.

`$ cp target/helloworld-mdb.war ../jboss-eap-7.1/standalone/deployment/`

* After a few seconds, you should see a message similar to the following appear in your EAP console log:

```
17:08:49,636 INFO  [org.jboss.as.repository] (DeploymentScanner-threads - 1) WFLYDR0001: Content added at location /Users/eapuser/Documents/product/jboss-eap-7.1/standalone/data/content/93/eba7b7c01c0525ad2bfcaab80fc120fcf08255/content
17:12:26,890 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-5) WFLYSRV0027: Starting deployment of "helloworld-mdb.war" (runtime-name: "helloworld-mdb.war")
17:12:27,263 INFO  [org.jboss.weld.deployer] (MSC service thread 1-3) WFLYWELD0003: Processing weld deployment helloworld-mdb.war
17:12:27,296 INFO  [org.hibernate.validator.internal.util.Version] (MSC service thread 1-3) HV000001: Hibernate Validator 5.3.5.Final-redhat-2
17:12:27,519 INFO  [org.infinispan.factories.GlobalComponentRegistry] (MSC service thread 1-4) ISPN000128: Infinispan version: Infinispan 'Chakra' 8.2.8.Final-redhat-1
17:12:27,543 INFO  [org.jboss.weld.Version] (MSC service thread 1-5) WELD-000900: 2.4.3 (redhat)
17:12:27,627 INFO  [org.jboss.as.ejb3] (MSC service thread 1-3) WFLYEJB0042: Started message driven bean 'HelloWorldQueueMDB' with 'activemq-ra-remote' resource adapter
17:12:27,788 INFO  [org.jboss.as.clustering.infinispan] (ServerService Thread Pool -- 70) WFLYCLINF0002: Started client-mappings cache from ejb container
17:12:28,359 INFO  [javax.enterprise.resource.webcontainer.jsf.config] (ServerService Thread Pool -- 10) Initializing Mojarra 2.2.13.SP4  for context '/helloworld-mdb'
17:12:28,714 INFO  [org.wildfly.extension.undertow] (ServerService Thread Pool -- 10) WFLYUT0021: Registered web context: '/helloworld-mdb' for server 'default-server'
17:12:28,766 INFO  [org.jboss.as.server] (DeploymentScanner-threads - 1) WFLYSRV0010: Deployed "helloworld-mdb.war" (runtime-name : "helloworld-mdb.war")
```
## Testing

* Open a web browser and navigate to `http://localhost:8080/helloworld-mdb`

* The browser should inform you of sent messages with the following output:

```
Sending messages to ActiveMQQueue[HELLOWORLDMDBQueue]

The following messages will be sent to the destination:
Message (0): This is message 1
Message (1): This is message 2
Message (2): This is message 3
Message (3): This is message 4
Message (4): This is message 5
Go to your JBoss EAP server console or server log to see the result of messages processing.
```

* In your EAP console log, you should see the following output:

```
17:13:19,311 INFO  [class org.jboss.as.quickstarts.mdb.HelloWorldQueueMDB] (Thread-2 (ActiveMQ-client-global-threads)) Received Message from queue: This is message 3
17:13:19,311 INFO  [class org.jboss.as.quickstarts.mdb.HelloWorldQueueMDB] (Thread-4 (ActiveMQ-client-global-threads)) Received Message from queue: This is message 5
17:13:19,310 INFO  [class org.jboss.as.quickstarts.mdb.HelloWorldQueueMDB] (Thread-3 (ActiveMQ-client-global-threads)) Received Message from queue: This is message 4
17:13:19,310 INFO  [class org.jboss.as.quickstarts.mdb.HelloWorldQueueMDB] (Thread-1 (ActiveMQ-client-global-threads)) Received Message from queue: This is message 2
17:13:19,310 INFO  [class org.jboss.as.quickstarts.mdb.HelloWorldQueueMDB] (Thread-0 (ActiveMQ-client-global-threads)) Received Message from queue: This is message 1
```



