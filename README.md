The solution / workaround
=========================

For me it looks like that the cxf-codegen-plugin still uses the Java 7 JDK to generate the code for a service. If you look at the genereted source of the Class HelloWorldService.java you can see that it uses a 3 parameter constructor of the super class. This construtor is available in the Java 7 rt.jar not in the Java 6 rt.jar.

The different between Java 6 & 7 is that Java 7 supports JAX-WS 2.2 and Java 6 only 2.1 and it looks like the constructor is only available in JAX-WS 2.2.

I added the following extraargs to the cxf-codegen-plugin to build JAX-WS 2.1 compatible code

```xml
<extraargs>
  <extraarg>-fe</extraarg>
  <extraarg>jaxws21</extraarg>
</extraargs>
```

This will fix the compile problem.

For me it is just a workaround and no solution because the plugin will still generate with Java 7. But it is workaround because the code is now compatible.


CXF codegen plugin is not toolchains aware
==========================================

See also [CXF-6268](https://issues.apache.org/jira/browse/CXF-6268) Make cxf-codegen-plugin toolchains aware during maven build

This project aims to point out that `cxf-codegen-plugin` is not _toolchains aware_.
The [maven-toolchains-plugins](http://maven.apache.org/plugins/maven-toolchains-plugin) provides to share configuration across plugins.

- this project enforces usage of JDK 1.7 or higher to run the `mvn` command
- it configures `maven-toolchains-plugins` to target JDK 6
- `cxf-codegen-plugin` doesn't read shared configuration from `maven-toolchains-plugin` so it will use jdk 7 API to generate source code
- the _compile_ phase will **fail** because JAXWS APIs provided by JDK 7 – used for source code generation – aren't backward compatibles with those provided by JDK 6 – used at compile time by `maven-compiler-plugin`.

How to run this example
-----------------------

- you must run maven commands with a JDK 7 or higher
- you must configure a toolchain _jdk_ version _6_ in your `~/.m2/toolchains.xml`
```xml
<toolchains>
  <toolchain>
    <type>jdk</type>
    <provides>
      <version>6</version>
    </provides>
    <configuration>
      <jdkHome>{path to you JDK 6 installation}</jdkHome>
    </configuration>
  </toolchain>
</toolchains>
```
- run `mvn generate-sources`
- take a look at generated source code in `target/generated-sources/cxf`
- you can see, for instance, that `HelloWorldService.java` uses a constructor which is not provided by JDK 6 
```
Service(java.net.URL,javax.xml.namespace.QName,javax.xml.ws.WebServiceFeature[])
```
See [javax.xml.ws.Service](http://docs.oracle.com/javase/6/docs/api/javax/xml/ws/Service.html).
- run `mvn compile` if you still have doubts
