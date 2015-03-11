---
layout: post
title:  Reference Guide
categories: SBP
weight: 300
parent: spring-data.html
---


{%summary%}{%endsummary%}

{%warning%}
This section of the documentation is under construction !
{%endwarning%}



# XAP Support

The first task to handle while developing XAP application using Spring would be to configure a connection to the active space inside the Spring IoC container. This part of the document will show you how basic connection can be applied using XML or Java based Spring configurations.

To get started, first you need to get the space running: either configure an Embedded Space or set up a Service Grid and deploy the Data Grid on it.

To use the Embedded Space you don't need to start any additional processes on your environment.

To start the Data Grid refer to [XAP Quick Start guide](http://docs.gigaspaces.com/xap100/your-first-data-grid-application.html) for an explanation on how to startup a space storage. Once installed, deploying Data Grid is typically a matter of executing the next commands from the `GS_HOME/bin` folder:

{%highlight console%}
*Windows*
gs-agent.bat
gs.bat deploy-space -cluster total_members=1,1 space

*Unix*
./gs-agent.sh
./gs.sh deploy-space -cluster total_members=1,1 space
{%endhighlight%}

Then in your project (assuming you build it with [Maven](http://maven.apache.org/)) add the following to `pom.xml` dependencies section:

{%highlight xml%}
<dependencies>
    <!-- other dependency elements omitted -->
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-xap</artifactId>
        <version>1.0.0.RELEASE</version>
    </dependency>
</dependencies>
{%endhighlight%}

# Connecting to space using XML based metadata

To use XAP Repository you need to provide a connection to space with an instance of `GigaSpace`. Basic access can be easily configured with next Spring XML configuration:

{%highlight xml%}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:os-core="http://www.openspaces.org/schema/core"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="
         http://www.openspaces.org/schema/core
         http://www.openspaces.org/schema/10.1/core/openspaces-core.xsd
         http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">

  <!-- A bean representing space proxy (requires active data grid with the same name) -->
  <os-core:space-proxy id="space" name="space"/>

  <!-- GigaSpace interface implementation used for SpaceClient injection -->
  <os-core:giga-space id="gigaSpace" space="space"/>

</beans>
{%endhighlight%}

If you want to use an Embedded Space, the next configuration will suite your needs:

{%highlight xml%}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:os-core="http://www.openspaces.org/schema/core"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="
         http://www.openspaces.org/schema/core
         http://www.openspaces.org/schema/10.1/core/openspaces-core.xsd
         http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">

  <!-- A bean representing an embedded space -->
  <os-core:embedded-space id="space" name="space"/>

  <!-- GigaSpace interface implementation used for SpaceClient injection -->
  <os-core:giga-space id="gigaSpace" space="space"/>

</beans>
{%endhighlight%}

# Connecting to space using Java based metadata

The same configuration can be achieved with next Java-based bean metadata:

{%highlight java%}
@Configuration
public class ContextConfiguration {
    /**
     * Builds a space instance with settings that allow it to connect to the 'space'.
     */
    @Bean
    public GigaSpace space() {
        UrlSpaceConfigurer urlSpaceConfigurer = new UrlSpaceConfigurer("jini://*/*/space");
        return new GigaSpaceConfigurer(urlSpaceConfigurer).gigaSpace();
    }
}
{%endhighlight%}

Or for the Embedded Space:

{%highlight java%}
@Configuration
public class ContextConfiguration {
    /**
     * Builds a space instance with settings that allow it start the embedded space with name 'space'.
     */
    @Bean
    public GigaSpace space() {
        UrlSpaceConfigurer urlSpaceConfigurer = new UrlSpaceConfigurer("/./space");
        return new GigaSpaceConfigurer(urlSpaceConfigurer).gigaSpace();
    }
}
{%endhighlight%}

# Other commonly used space configurations

In some projects you might want to apply other configurations to your space. There are a lot of options available to you.

## Local cache

A Local Cache is a Client Side Cache that maintains a subset of the master space’s data based on the client application’s recent activity. The local cache is created empty, and whenever the client application executes a query the local cache first tries to fulfill it from the cache, otherwise it executes it on the master space and caches the result locally for future queries.

To find out more about Local Cache and it's configuration, please, proceed to [Local Cache Reference]({%latestjavaurl%}/local-cache.html).

## Local view

A Local View is a Client Side Cache that maintains a subset of the master space’s data, allowing the client to read distributed data without performing any remote calls or data serialization. Data is streamed into the client local view based on predefined criteria (a collection of SQLQuery objects) specified by the client when the local view is created.

To read more on Local View, please, proceed to [Local View Reference]({%latestjavaurl%}/local-view.html).

## Space Persistence

A Space Persistence is a configuration where space data is persisted into permanent storage and retrieved from it.

Read more on persisting the space data [here]({%latestjavatuturl%}/java-tutorial-part7.html).

## Event Processing

You might want to organize your system into event-driven structure. Using XAP there are several native features that will help you achieve your goals, among them: [Notify Container]({%latestjavaurl%}/notify-container-overview.html), [Polling Container]({%latestjavaurl%}/polling-container-overview.html), [Archive Container]({%latestjavaurl%}/archive-container.html).

## Space Security

XAP Security provides comprehensive support for securing your data, services, or both. XAP provides a set of authorities granting privileged access to data, and for performing operations on services.

To find out more about security configuration, please, refer to [Security Guide](http://docs.gigaspaces.com/xap101sec/).

# Using native write and read operation

`GigaSpace` configured above can be used directly to perform interaction with space. To do so, you can simply inject `GigaSpace` bean into your Repository classes. Let's see an example of such usage. First, here is an example of POJO class:

{%highlight java%}
@SpaceClass
public class Person {
    private Integer id;
    private String name;
    private Integer age;

    public Person() {
    }

    @SpaceId(autoGenerate = true)
    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    // getters and setters for other fields are omitted

}
{%endhighlight%}

{%note%}
The class is marked with `@SpaceClass` annotation - it allows Spring XAP to look for entities in your data model and automatically handle their structure. Also, the `getId()` method is marked with `@SpaceId(autogenerate = true)` annotation - it will tell the space to handle ids automatically.
{%endnote%}

Now `GigaSpace` can be injected and used directly in Repository layer:

{%highlight java%}
@Repository
public class XapPersonRepository implements PersonRepository {

    @Autowired
    private GigaSpace space;

    public void create(Person person) {
        space.write(person);
    }

    public List<Person> findById(String personId) {
        return space.readById(Person.class, personId);
    }
    ...
}
{%endhighlight%}

# Modeling your data

Spring Data XAP comes with the transparent support of XAP native features. Among them, some additional configuration can be applied to your POJOs to boost up the performance, reduce memory usage or just ease the model understanding. When building data model using Spring Data XAP you might want to pay attention to the next features:

##  Indexing

The most well-known data store function that allows to boost up common queries performance is index support. XAP provides several options here: basic, compound and unique indexes. All of these features can be applied by simply annotating POJO classes or their fields, e.g. with `@SpaceIndex` or `@CompoundSpaceIndex` annotations. Please, refer to [Indexing]({%latestjavaurl%}/indexing-overview.html) for more details and examples of POJO classes.

## Storage Types

You can define the form in which objects will be stored in Space either with annotations on each POJO in your model or with defining default Storage Type for the whole Space. This is done to save up time on serialization/de-serialization, reduce memory usage or to define schema that will change in time. Three Storage Types are available for POJOs: `OBJECT`, `BINARY` and `COMPRESSED`. Again, whole configuration can be applied just by annotation your model classes. To read more on this feature, please, refer to [Storage Types](http://docs.gigaspaces.com/xap101/storage-types---controlling-serialization.html).

## Exclusion

You can mark some POJO properties with `@SpaceExclude` to disable writing their values to the Space. It will also affect Querydsl `Q...` classes generation from POJOs - marked fields won't be available for querying in Querydsl style.

##  Other Annotation-based Features

There are lots of other useful annotation-based configuration possibilities available to you. For the full list of them, please, refer to [Annotation based Metadata]({%latestjavaurl%}/pojo-annotation-overview.html).