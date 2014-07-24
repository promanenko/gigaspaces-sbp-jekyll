---
layout: post
title:  XAP Websphere Dynamic Caching
categories: SBP
parent: data-access-patterns.html
weight: 30000
---

{%summary%}{%endsummary%}

{% tip %}
**Summary:** {% excerpt %}TODO{% endexcerpt %}<br/>
**Author**: Shay Hassidim, Deputy CTO, GigaSpaces<br/>
**Recently tested with GigaSpaces version**: TODO <br/>


{% endtip %}

# Overview

TODO



# Requirements

The following technology is required to run this demo

An Application Server from the WebSphere Family <br>
- WebSphere Application Server 7+  <br>
- WebSphere Enterprise Service Bus <br>
- IBM Integration Bus         <br>
- IBM WebSphere Process Server


An IBM IDE <br>
- Ration Application Developer <br>
- IBM Integration Designer <br>

GigaSpaces Software <br>
- GigaSpaces XAP Premium Edition 9+ <br>
- GigaSpacesDynaCacheIntegration.jar<br>
- GigaDynaCacheTestWeb.war


# Create New Dynamic Cache Resource

Start your Websphere Application Server

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache1.png)

Open the Administrative Console
![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache2.png)

Login to the administrative console

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache3.png)

Expand “Resources” -> “Cache instances” and Click on “Object cache instances”

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache4.png)

Choose the “Node” scope from the drop-down list

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache5.png)

Click the “New” button

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache6.png)

Enter the name and JNDI specified below. Then click “OK” and “Save”

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache7.png)

Click on “Demo Cache Instance”
![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache8.png)

Click on “Custom properties”

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache9.png)

Add new property “com.ibm.ws.cache.CacheConfig.showObjectContents=true”

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache10.png)

Restart WebSphere application server

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache11.png)


# Run Demo with Dynamic Cache as WebSphere Caching Provider

### Deploy Dynamic Cache Monitor

Expand “Applications” -> “Application Types” and Click on “Websphere enterprise applications”.
On the “Enterprise Applications” screen, click the “Install” button

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache12.png)

Browse into your Websphere installation /AppServer/installableApps, Click on CacheMonitor.ear

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache13.png)

Download IBM Extended Cache Monitor from IBM’s offical site.
Extract the contents of cachemonitor7_package into any temp directory or onto your desktop

![imdg_eviction_large_db.jpg](/sbp/attachment_files/ibm-cache14.png)
























