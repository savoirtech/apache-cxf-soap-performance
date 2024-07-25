# Apache CXF SOAP Performance

You know what would be fun? Running 1 Billion soap invocations through
CXF!

# The Setup

<figure>
<img src="./assets/images/HardwareSetup.png" alt="POWER" />
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
<img src="./assets/images/Apache-CXF-Perf-Harness.png" alt="POWER" />
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
<img src="./assets/images/SoapInvocations.png" alt="POWER" />
</figure>

Once the time duration has been met, it will cease the executing
clients, and tabulate the total invocations.

# Lets get this test case running

To run the performance harness we change directory into
samples/performance. Within this folder we’ll build the base harness and
the various scenarios.

``` bash
$ cd samples/performance
$ mvn clean install
```

For the purposes of our lab test, we’ll configure the test to use 100
clients, over a period of 8 hours (60 x 60 x 8 = 28800 seconds).

# Results

``` bash
=============Overall Test Result============
Overall Throughput: echoComplexTypeDoc ??? (invocations/sec)
Overall AVG. response time: ??? (ms)
??? (invocations), running 2880000 (sec)
============================================
```

# Conclusion

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
