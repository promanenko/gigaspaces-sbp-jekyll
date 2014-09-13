---
layout: post
title:  XAP Websphere Dynamic Caching
categories: SBP
parent: data-access-patterns.html
weight: 60
---

{%summary%}{%endsummary%}

{% tip %}
**Summary:**{% excerpt %}This article illustrates how to integrate IBM's DynaCache with GigaSpaces XAP{% endexcerpt %}<br/>
**Author**: Allen Terleto<br/>
**Recently tested with GigaSpaces version**: XAP 9.7<br/>
**Last Update:** September 2014<br/>

{% endtip %}



# Introduction
IBM provides an embedded cache called Dynamic Cache (DynaCache) as a feature of their WebSphere Application Server. By utilizing a caching-strategy applications can bypass the latency-costs of processing web services, business logic, data access, network & IO overhead, etc. DynaCache is the underlying caching strategy for many of the WebSphere-based family of products such as:

•	WebSphere Commerce Server <br>
•	WebSphere Portal Server  <br>
•	WebSphere Enterprise Service Bus <br>
•	Business Process Manager    <br>
•	WebSphere Application Server

WebSphere allows administrators to configure the Dynamic Cache Service to use GigaSpace’s XAP IMDG as its alternative Cache Provider instead of the default DynaCache. GigaSpaces XAP enhances WebSphere’s caching strategy by providing:

•	Elastic Scalability <br>
•	High availability <br>
•	Transactional Support <br>
•	Event Processing <br>
•	Persistency <br>
•	Filtering   <br>
•	Big Data Integration  <br>
•	Security       <br>
•	Enhanced Monitoring Capabilities <br>
•	Distributed and Centralized System of Record  <br>
•	HTTP Session Sharing (across various Web Containers) <br>
•	Replication across multiple Data Centers<br>
•	Other XAP Features…

GigaSpace’s XAP features increase the capabilities of WebSphere’s DynaCache beyond the limitations of the default dynamic cache engine and data replication service. While DynaCache can only provide caching across replicated and synchronized WebSphere application servers, GigaSpaces XAP provides a truly distributed and remote caching architecture. WebSphere administrators no longer need to worry about data loss due to cluster failures, redeployments and upgrades. Additionally, scaling the Data Cache Tier will no longer be a function of adding additional WebSphere Application Server instances.

If you are currently using DynaCache, you can simply use the administrative console or wsadmin commands to replace WebSphere’s default cache provider with GigaSpace’s XAP. You do not have to make any changes to the code interacting with the default dynamic cache or caching data model. In the Demo: Step-By-Step Walkthrough section we will show how to switch to GigaSpace’s XAP IMDG in just a few configuration changes. Please read the GigaSpaces XAP Integration with IBM Dynamic Cache for a detailed walkthrough and additional introduction information.

{%comment%}
# Use cases

### In-Line Cache
With this mechanism, the IMDG is the system of record. The database data is loaded into the IMDG when it is started. The IMDG is responsible for loading the data and pushing updates back into the database. The database can be updated in synchronously or asynchronously.

• When running in all-in-cache cache policy mode, all data is loaded from the database into the cache once it is started.

• When running in LRU cache policy mode, a subset of the data is loaded from the database into the cache when it is started. Data is evicted from the cache based on available memory or a maximum amount of cache objects. Once there is a cache miss, the cache looks for the data within the underlying data-source. If matching data is found, it is loaded into the cache and delivered to the application.

![drools1](/sbp/attachment_files/dynacache/cache1.png)


### HTTP Session Sharing
It’s becoming increasingly important for organizations to share HTTP session data across multiple data centers, multiple web server instances or different types of web servers. Here are a few scenarios where HTTP session sharing is required:

•	Multiple different Web Servers running your Web Application - You may be porting your application from one web server to another and there will be a period of time when both types of servers need to be active in production.

•	Web Application is broken into multiple modules - When applications are modularized such that different functionalities are deployed across multiple server instances. For example, you may have login, basic info, check-in and shopping functionalities split into separate modules and deployed individually across different servers for manageability or scalability. In order for the user to be presented with a single, seamless, and transparent application, session data needs to be shared between all the servers.

•	Reduce Web application memory footprint - The web application storing all sessions within the web application process heap is consuming large amounts of memory. Having the session stored within a remote process will reduce web application utilization avoiding garbage collocation and long pauses.

•	Multiple Data-Center deployment - You may need to deploy your application across multiple data centers for high-availability, scalability or flexibility, so session data will need to be replicated.

The following diagram depicts a common use case where there are multiple data centers connected across the WAN, and each is running a different type of web server.

![drools1](/sbp/attachment_files/dynacache/cache2.png)

### WebSphere Commerce Server Performance
The WebSphere Commerce Server is an IBM e-commerce framework that is widely used in retail environments of many sizes. The WebSphere Commerce Server contains the components for the various B2B and B2C functionalities needed in a retail environment.

WebSphere Commerce Server applications heavily leverage DynaCache to reduce database roundtrips and thus gain improved performance in terms of latency. WebSphere Commerce uses DynaCache to store entire JSP(s), page fragments, and commands. While DynaCache stores portions of its data in the available heap space of the clustered WebSphere Commerce Servers, the remaining overflow of data persists to disk. By using GigaSpace’s XAP as an alternative caching provider, the WebSphere Commerce Server applications are not captive to the limited shared-memory of the server cluster nor will they pay the penalty of overflowing their data to disk.

E-Commerce web sites aspiring to manage high volumes of traffic might find that the default caching provider of the WebSphere Commerce Server will not scale to meet SLA requirements. With only a few simple configurations changes this bottleneck can be removed by using GigaSpace’s XAP as the alternative caching provider.

{%endcomment%}

# Demo

### Requirements

The following technology is required to run this demo

An Application Server from the WebSphere Family <br>
•	WebSphere Application Server 7+    <br>
•	WebSphere Enterprise Service Bus  <br>
•	IBM Integration Bus  <br>
•	IBM WebSphere Process Server

### An IBM IDE<br>
•	Rational Application Developer<br>
•	IBM Integration Designer

### GigaSpaces Software <br>
•	GigaSpaces XAP Premium Edition 9+ <br>
•	GigaSpacesDynaCacheIntegration.jar <br>
•	GigaDynaCacheTestWeb.war




### Create New Dynamic Cache Resource

Enter the name and JNDI specified below
![drools1](/sbp/attachment_files/dynacache/cache3.png)


Add new property “com.ibm.ws.cache.CacheConfig.showObjectContents=true”
![drools1](/sbp/attachment_files/dynacache/cache4.png)



### Deploy Dynamic Cache Monitor
Expand “Applications” -> “Application Types” and Click on “Websphere enterprise applications”.
On the “Enterprise Applications” screen, click the “Install” button

![drools1](/sbp/attachment_files/dynacache/cache5.png)



Browse into your Websphere installation /AppServer/installableApps, Click on CacheMonitor.ear

![drools1](/sbp/attachment_files/dynacache/cache6.png)



### Download IBM Extended Cache Monitor from IBM’s offical site.
Extract the contents of cachemonitor7_package into any temp directory or onto your desktop

![drools1](/sbp/attachment_files/dynacache/cache7.png)

In your admin console, click the checkbox near “Dynamic Cache Monitor” and Click “Update”

![drools1](/sbp/attachment_files/dynacache/cache8.png)


Choose “Replace, add, or delete multiple files” and browse for “cachemonitor7_update.zip”

![drools1](/sbp/attachment_files/dynacache/cache9.png)



### Setup Dynamic Cache Monitor Security

![drools1](/sbp/attachment_files/dynacache/cache10.png)


Open an Internet Browser and go to http://localhost:9082/cachemonitor
Enter your User ID and Password then Click OK.

![drools1](/sbp/attachment_files/dynacache/cache11.png)



Find the Object Cache you created earlier “cache/demo” and Click OK.

![drools1](/sbp/attachment_files/dynacache/cache12.png)

### Deploy Demo Application

Expand “Applications” -> “Application Types” and Click on “Websphere enterprise applications”.
On the “Enterprise Applications” screen, click the “Install” button

![drools1](/sbp/attachment_files/dynacache/cache13.png)



Browse for the “GigaDynaCacheTestWeb.war” in your local directory and Click “Next”.
Continue to Click “Next” until the summary page. Then click “Finish” and “Save”

![drools1](/sbp/attachment_files/dynacache/cache14.png)


### Test Demo Application using Dynamic Cache

Open an Internet Browser and go to http://localhost:9082/GigaDynaCacheTestWeb/DynaCacheTestServlet

![drools1](/sbp/attachment_files/dynacache/cache15.png)


Go back to the Cache Monitor Page and click “Refresh Statistics”
![drools1](/sbp/attachment_files/dynacache/cache16.png)



### Run Demo with GigaSpaces’ XAP as WebSphere Caching Provider

Add GigaSpaces Jars to IBM Extension Classloader

Scroll down to find the Server Infrastructure Section. Expand “Java and Process Management”.
Then click on “Process definition”

![drools1](/sbp/attachment_files/dynacache/cache17.png)



Click on “Java Virtual Machine”
![drools1](/sbp/attachment_files/dynacache/cache18.png)


Add a new custom properties pointing to the directory with the XAP jars  “ws.ext.dirs= C:\Gigaspaces\gigaspaces-xap-premium-9.7.0-ga\lib\required;C:\temp\custom\GigaSpacesDynaCacheIntegration.jar”

![drools1](/sbp/attachment_files/dynacache/cache19.png)

### Configure GigaSpaces’ XAP as Alternative WebSphere Caching Provider

Choose “GigaSpaces XAP” from the dropdown choices of “Cache provider”. Click “OK” and “Save”

![drools1](/sbp/attachment_files/dynacache/cache20.png)

Click on custom properties and add the following:<br>
•	com.ibm.ws.cache.CacheConfig.cacheProviderName=com.ibm.ws.objectgrid.dynacache.CacheProviderImpl  <br>
•	xap.space.url=jini://*/*/mySpace?groups=myGroup


![drools1](/sbp/attachment_files/dynacache/cache21.png)

### Start XAP Runtime Environment

Run gs-agent from your XAP bin directory

![drools1](/sbp/attachment_files/dynacache/cache22.png)

Run gs-ui from the same directory

![drools1](/sbp/attachment_files/dynacache/cache23.png)



Deploy a new Data Grid called “mySpace” with any SLA

![drools1](/sbp/attachment_files/dynacache/cache24.png)

Confirm that there are no entries in the cache using the “Space Browser” tab

![drools1](/sbp/attachment_files/dynacache/cache25.png)

### Test Demo using XAP as Caching Provider

Open an Internet Browser and go to http://localhost:9082/GigaDynaCacheTestWeb/DynaCacheTestServlet

![drools1](/sbp/attachment_files/dynacache/cache26.png)

Check the gs-ui to confirm that a new entry is in the Space

![drools1](/sbp/attachment_files/dynacache/cache27.png)

















