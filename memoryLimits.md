---

copyright:
  years: 2015, 2017
lastupdated: "2017-07-14"

---

{:new_window: target="_blank"}
{:codeblock: .codeblock}

# Memory limits and the Liberty buildpack
{: #memory_limits}

A memory limit must be specified when you deploy an application with the Liberty buildpack.

## Memory limits and the Liberty buildpack
{: #memory_limits_and_liberty}


When you deploy an application with the Liberty buildpack, you are prompted for a "Memory Limit". The Liberty for Java buildpack sets a default heap memory size ratio to prevent your process from exceeding the memory limit. You can also specify the heap memory size or ratio by defining the `JVM_ARGS` or `JBP_CONFIG_IBMJDK` environmental variables. See [Heap memory options](#heap_memory_options) to learn more.

To determine what Memory Limit to specify, it is important to understand that you are not specifying the size of the Java application heap. You are specifying the amount of memory available for the Diego container or DEA to use. This amount includes the memory that is used by the following components:

* by the warden container.
* to map kernel extensions and system libraries into the process.
* for the jvm executables, jvm working heap, jit space, and compressed references.
* for classes (application server and application), thread stacks, and direct io buffers.
* by the Java application heap.

More information on JVM memory usage can be found at the developerWorks article [Thanks for the memory](http://www.ibm.com/developerworks/library/j-nativememory-linux/)

When you deploy an application, the memory usage of the entire process is monitored. If memory usage exceeds the memory limit that you specified when the application was deployed, the kernel stops the process. This action happens without warning and might be manifested in several ways.

 If the Memory Limit is exceeded during application deployment, you receive a message that a failure occured and you might see any the following.

  * You might see that the application is flapping.
  * You might see that the application attempted to start multiple times, always unsuccessfully.
  * You might receive a message that indicates the application deployment failed.

If the Memory Limit is exceeded while the application is in service, the process stops and Cloud Foundry attempts to restart the application. The application might restart, but it is unavailable for some amount of time.

## Heap memory options
{: #heap_memory_options}

You can customize the maximum amount of heap memory your application is allocated in various ways including by using the `JVM_ARGS` environment variable, changing the `jvm.options` file, or setting the `JBP_CONFIG_IBMJDK` environment variable which controls the `heap_size_ratio`. If you do not specify any settings, the buildpack uses the default settings.

### Heap memory defaults
{: .#heap_memory_defaults}
To prevent errors that result from exceeding mamory limits, the Liberty for Java buildpack sets a default heap size ratio depending on the memory limit that you specify when you deploy your application.

* Applications with memory limits of less than 512M have a heap size ratio of 50%
* Applications that have memory limits greater than or equal to 512M have a heap size ratio of 75%

When you specify the heap memory using environment variables, you override the default heap size ratios.

### Specifying Heap Memory
{: #specifying_heap_memory}

You can set the heap memory size by using environment variables or by changing the `jvm.options` file. When you use either the `JVM_ARGS` or `JBP_CONFIG_IBMJDK` environment variables, any changes will take effect when pushing your application to Bluemix. By changing the `jvm.options` file, the effect to the heap size configuration can also be tested locally.

* Use the `JVM_ARGS` environment variable and the -Xmx argument. For example to set the maximum heap size to 512M use the command that follows, then restage your app.

```
    $ cf set-env myapp JVM_ARGS -Xmx512m
```
{: codeblock}

* Specify the heap size ratio using the JBP_CONFIG_IBMJDK environment variable.  The heap_size_ratio is a floating point value which specifies how much of memory limit to allocate to the heap.  For example, to allocate half of the available memory to the heap (50% or 0.50), issue the following command and restage your app.

```
    $ cf set-env myapp JBP_CONFIG_IBMJDK "heap_size_ratio: 0.50"
```
{: codeblock}

* Specify the -Xmx argument in the jvm.options file if your application is a [server directory](optionsForPushing.html#server_directory) or a [packaged server](optionsForPushing.html#packaged_server). For more inofrmation on usig the `jvm.options` file with the Liberty runtime, see [Setting generic JVM](http://www-01.ibm.com/support/docview.wss?uid=swg21596474).  
