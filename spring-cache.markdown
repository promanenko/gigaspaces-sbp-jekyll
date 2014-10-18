---
layout: post
title:  Spring Cache Abstraction with XAP
categories: SBP
parent: data-access-patterns.html
weight: 101
---

{% mute %}This article shows how to  use the Spring Cache Abstraction Provider with  XAP{% endmute %}

{% panel %}
{%section%}
{%column width=15% %}
**XAP Version**<br>
**Last Updated**<br>
**Reference**<br>
**Example**
{%endcolumn%}
{%column  width=50% %}
9.7<br>
October 2014<br>
[Spring Framework](http://docs.spring.io/spring-framework/docs/4.0.x/spring-framework-reference/html/cache.html)<br>
{%git https://github.com/GigaSpaces-ProfessionalServices/spring-cache-abstraction %}
{%endcolumn%}
{%column  width=10% %}
{%align right%}
**Author**
{%endalign%}
{%endcolumn%}
{%column  width=25% %}
{%align center%}
Ali Hodroj <br>
Director of Solution Architecture<br>
GigaSpaces
{%endalign%}
{%endcolumn%}
{%endsection%}
{% endpanel %}


{%summary%}{%endsummary%}


# Introduction

Since version 3.1, the `Spring Framework` provides support for transparently adding caching to an existing `Spring` application through method annotations. As is the case with transaction support, the caching abstraction decouples caching implementation from the business logic. This article shows how to utilize the XAP CacheManager with Spring that implements the following distributed caching scenario:


{: .table .table-bordered .table-condensed}
|------|-----|
|**Distributed cache**|**Distributed cache + local cache** |
|![scaling_agent.jpg](/sbp/attachment_files/spring-cache1.png)|![scaling_agent.jpg](/sbp/attachment_files/spring-cache2.png)|



{%vbar title=Benefits when using XAP as a Spring caching provider: %}

-	**Decreased Latency** –  Ability to utilize local cache across all server instances for localized reads, greatly reducing serialization across the wire

-	**Scalability** – Horizontally scalable and partitioned cache with flexible cache entry routing

-	**High Availability** – Each partition has a backup instance deployable on a different JVM, host, or availability zone.
{%endvbar%}




# Configuring the Cache Storage

The XAP cache manager is located under `org.openspaces.cacheable` package. To use it, you simply declare the `GigaSpacesCacheManager` as the cacheManager bean. In addition, the “space” property must be set to specify the URI of the XAP cache.

{%highlight xml%}
<cache:annotation-driven />
<bean id="cacheManager" class="org.openspaces.cacheable.GigaSpacesCacheManager">
    <property name="space" value="jini://*/*/space" />
</bean>
{%endhighlight%}

## Declarative Annotation-based Caching

Spring provides two annotations for caching declaration and eviction: `@Cacheable` and `@CacheEvict`. The `@Cacheable` annotation will trigger a cache put, while `@CacheEvict` will clear all space entries associated with a given cache:

{%highlight java%}
@Cacheable("books")
public Book findBook(ISBN isbn) {...}

@CacheEvict(value = "books", allEntries=true)
public void loadBooks(InputStream batch)
{%endhighlight%}

## Data-locality with a local cache (highest performance)

Since most cache queries in Spring are ID-based lookups, the performance can be dramatically improved by utilizing the [Local Cache]({%latestjavaurl%}/local-cache.html) feature of XAP.

Here is an example how to configure the Local Cache:

{%highlight xml%}
<cache:annotation-driven />
<bean id="cacheManager" class="org.openspaces.cacheable.GigaSpacesCacheManager">
   <property name="space" value="jini://*/*/space" />
   <property name="localCache" value="true" />
</bean>
{%endhighlight%}



