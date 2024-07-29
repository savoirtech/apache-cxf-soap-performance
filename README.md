# Apache CXF SOAP Performance

You know what would be fun? Attempting to run 1 Billion soap invocations
through CXF in 8 hours!

# The Setup

<figure>
<img src="./assets/images/HardwareSetup.png" alt="Hardware" />
</figure>

For our lab test we’ll be using the following hardware:

- Dell PowerEdge R250 (Client Host)

  - Intel Xeon E-2378 (8c, 16t)

  - 128 GB DDR4 RAM

  - 1 Gigabit Ethernet

- Raptor Blackbird (Server Host)

  - POWER9 (8c, 32 t)

  - 128 GB DDR4 RAM

  - 1 Gigabit Ethernet

The machines are co-located on the same switch, reducing the number of
packet hops.

# The Performance Harness

As of CXF 4.1 the binary distribution will contain a set of performance
scripts in the samples folder. Options to test JAX-WS and JAX-RS are
present.

<figure>
<img src="./assets/images/Apache-CXF-Perf-Harness.png" alt="Perf" />
</figure>

At its core, the performance harness is a client-server request/response
automation. On startup the script initializes and warms up the JVM for
executing mass calls.

## How it works

The client host runs a number of threads, each running a CXF client
which invokes up the server host. For JAX-WS testing, we have a choice
of invoking on DocPortType as String, Base64, or as a ComplexType. The
client side harness will run N threads for M invocations for a duration
of time.

<figure>
<img src="./assets/images/SoapInvocations.png" alt="Soap" />
</figure>

Once the time duration has been met, it will cease the executing
clients, and tabulate the total invocations.

# Lets get this test case running

To run the performance harness we change directory into samples. Within
this folder we’ll build the base harness and the various scenarios.

On each host we will open a terminal to the CXF distribution samples
folder.

We’ll ensure we have JAVA_HOME and MAVEN_HOME environment variables set.

For our first run we’ll use Adoptium Eclipse Temurin 17 LTS as Client
and Server side JVM.

``` bash
$ cd samples
$ mvn clean install
$ cd performance/soap_http_doc_lit
```

On the Server host we’ll execute the following maven profile:

``` bash
$mvn -Pserver -Dhost=0.0.0.0 -Dprotocol=http
```

On the Client host we’ll execute the client profile, supplying
instructions to use HTTP protocol, echoComplexTypeDoc operation, use 100
threads (simulate 100 clients), over a time of 8 hours (60 x 60 x 8 =
28800 seconds).

``` bash
$mvn -Pclient -Dhost=192.168.50.154 -Dprotocol=http -Doperation=echoComplexTypeDoc -Dthreads=100 -Dtime=28800
```

For the purposes of our lab test, we’ll allow the suite to execute
without added agents to the JVM.

# LAB TIME!

## *First Run*

Our first iteration of this test resulted in 953,428,857 invocations in
8 hours…​ 47 Million less than our goal of 1 Billion.

``` bash
=============Overall Test Result============
Overall Throughput: echoComplexTypeDoc 331.05150369031566 (invocations/sec)
Overall AVG. response time: 3.0206780179299737 (ms)
9.53428857E8 (invocations), running 2880001.5900000003 (sec)
============================================
```

## *Second Run*

On our second run we re-configure our JAVA_HOME to use Adoptium Eclipse
Temurin 21 LTS and pass in the following arguments to the JVMs:

``` bash
$mvn -Pserver -Dhost=0.0.0.0 -Dprotocol=http
```

``` bash
$mvn -Pclient -Dhost=192.168.50.154 -Dprotocol=http -Doperation=echoComplexTypeDoc -Dthreads=100 -Dtime=28800
```

This resulted in:

``` bash
=============Overall Test Result============
Overall Throughput: echoComplexTypeDoc ??? (invocations/sec)
Overall AVG. response time: ??? (ms)
??? (invocations), running ??? (sec)
============================================
```

## *Third Run*

The first two test iterations used default CXF and JVM properties. On
our third run lets pass in a simple CXF configuration to Client and
Server JVMs to instruct them enable HTTP2 & AsyncHTTPConduit.

client.xml

``` xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:cxf="http://cxf.apache.org/core"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://cxf.apache.org/core
    http://cxf.apache.org/schemas/core.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <cxf:bus>
        <cxf:properties>
            <entry key="use.async.http.conduit" value="true"/>
            <entry key="org.apache.cxf.transports.http2.enabled" value="true"/>
        </cxf:properties>
    </cxf:bus>

</beans>
```

server.xml

``` xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:cxf="http://cxf.apache.org/core"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://cxf.apache.org/core
    http://cxf.apache.org/schemas/core.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <cxf:bus>
        <cxf:properties>
            <entry key="use.async.http.conduit" value="true"/>
            <entry key="org.apache.cxf.transports.http2.enabled" value="true"/>
        </cxf:properties>
    </cxf:bus>

</beans>
```

Before we can execute this iteration, we’ll need to update the pom.xml
to include a few more dependencies:

``` xml
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-transports-http-jetty</artifactId>
    <version>${project.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>6.1.11</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.1.11</version>
</dependency>
<dependency>
    <groupId>org.eclipse.jetty.http2</groupId>
    <artifactId>jetty-http2-server</artifactId>
    <version>${cxf.jetty12.version}</version>
</dependency>
```

Reusing Adoptium Eclipse Temurin 21 LTS, we pass the following arguments
to the JVMs:

``` bash
$mvn -Pserver -Dhost=0.0.0.0 -Dprotocol=http -Dcfg=client.xml
```

``` bash
$mvn -Pclient -Dhost=192.168.50.154 -Dprotocol=http -Doperation=echoComplexTypeDoc -Dthreads=100 -Dtime=28800 -Dcfg=server.xml
```

When the server side starts up, we’ll observe h2c in the logging.

``` bash
INFO: Started ServerConnector@9f73b40{HTTP/1.1, (http/1.1, h2c)}{localhost:8080}
```

# Results and Conclusion

Lets recap:

| Iteration | JVM | Throughput (invocations/sec) | Total Invocations in 8 Hours |
|----|----|----|----|
| 1 | Adoptium 17 | 331.0 | 953,428,857 |
| 2 | Adoptium 21 | ??? | ??? |
| 3 | Adoptium 21 | ??? | ??? |

# About the Authors

[Jamie
Goodyear](https://github.com/savoirtech/blogs/blob/main/authors/JamieGoodyear.md)

# Reaching Out

Please do not hesitate to reach out with questions and comments, here on
the Blog, or through the Savoir Technologies website at
<https://www.savoirtech.com>.

# With Thanks

Thank you to the Apache CXF community.

\(c\) 2024 Savoir Technologies
