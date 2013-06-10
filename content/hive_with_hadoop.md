---
title: Hive with Hadoop
---

Hive Integration
----------------

When working with [hive.apache.org](http://hive.apache.org) from a Java environment, one can choose between the Thrift client or using the Hive JDBC-like driver. Both have their pros and cons but no matter the choice, Spring and SHDP supports both of them.

###Starting a Hive Server

SHDP provides a dedicated namespace element for starting a Hive server as a Thrift service (only when using Hive 0.8 or higher). Simply specify the host, the port (the defaults are localhost and 10000 respectively) and you're good to go:

```bash
<!-- by default, the definition name is 'hive-server' -->
<hdp:hive-server host="some-other-host" port="10001" />
```

If needed the Hadoop configuration can be passed in or additional properties specified. In fact hiver-server provides the same properties configuration knobs as hadoop configuration:

```bash
<hdp:hive-server host="some-other-host" port="10001" properties-location="classpath:hive-dev.properties" configuration-ref="hadoopConfiguration">
  someproperty=somevalue
  hive.exec.scratchdir=/tmp/mydir
</hdp:hive-server>
```

The Hive server is bound to the enclosing application context life-cycle, that is it will automatically startup and shutdown along-side the application context.

###Using the Hive Thrift Client

Similar to the server, SHDP provides a dedicated namespace element for configuring a Hive client (that is Hive accessing a server node through the Thrift). Likewise, simply specify the host, the port (the defaults are localhost and 10000 respectively) and you're done:

```bash
<!-- by default, the definition name is 'hiveClientFactory' -->
<hdp:hive-client-factory host="some-other-host" port="10001" />
```

Note that since Thrift clients are not thread-safe, hive-client-factory returns a factory (named org.springframework.data.hadoop.hive.HiveClientFactory) for creating HiveClient new instances for each invocation. Further more, the client definition also allows Hive scripts (either declared inlined or externally) to be executed during initialization, once the client connects; this quite useful for doing Hive specific initialization:

```bash
<hive-client-factory host="some-host" port="some-port" xmlns="http://www.springframework.org/schema/hadoop">
   <hdp:script>
     DROP TABLE IF EXITS testHiveBatchTable; 
     CREATE TABLE testHiveBatchTable (key int, value string);
   </hdp:script>
   <hdp:script location="classpath:org/company/hive/script.q">
       <arguments>ignore-case=true</arguments>
   </hdp:script>
</hive-client-factory>
```

In the example above, two scripts are executed each time a new Hive client is created (if the scripts need to be executed only once considering using a tasklet) by the factory. The first executed a script defined inlined while the second read the script from the classpath and passed one parameter to it. For more information on using parameter (or variables) in Hive scripts, see this section in the Hive manual.