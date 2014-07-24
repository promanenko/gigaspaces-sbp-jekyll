---
layout: post
title:  Global HTTP Session Sharing
categories: SBP
parent: solutions.html
weight: 200
---


{%summary%}{%endsummary%}


{% note %}
XAP 10.0 Global HTTP Session Sharing includes the following new features:

1. Delta update support – changes identified at the session attribute level.
2. Better serialization (Kryo instead of xstream)
3. Compression support
4. Principle / Session ID based session management. Allows session sharing across different apps with same SSO
5. Role based SSO Support
6. Better logging

{% endnote %}

{% info title=Licensing %}
This feature requires a separate license in addition to the XAP commercial license. Please contact [GigaSpaces Customer Support](http://www.gigaspaces.com/content/customer-support-services) for more details.
{% endinfo %}



# Overview


It's becoming increasingly important for organizations to share HTTP session data across multiple data centers, multiple web server instances or different types of web servers. Here are few scenarios where HTTP session sharing is required:

- **Multiple different Web servers running your web application** - You may be porting your application from one web server to another and there will be a period of time when both types of servers need to be active in production.
- **Web Application is broken into multiple modules** - When applications are modularized such that different functionalities are deployed across multiple server instances. For example, you may have login, basic info, check-in and shopping functionalities split into separate modules and deployed individually across different servers for manageability or scalability. In order for the user to be presented with a single, seamless, and transparent application, session data needs to be shared between all the servers.
- **Reduce Web application memory footprint** - The web application storing all session within the web application process heap, consuming large amount of memory. Having the session stored within a remote process will reduce web application utilization avoiding garbage collocation and long pauses.
- **Multiple Data-Center deployment** - You may need to deploy your application across multiple data centers for high-availability, scalability or flexibility, so session data will need to be replicated.

[Global Http Session Sharing](http://www.slideboom.com/presentations/631622/Global-Http-Session-Sharing-V2)

{%comment%}
[Global Http Session Sharing](http://www.slideboom.com/presentations/631622/Global-Http-Session-Sharing-V2)


<object classid="clsid:d27cdb6e-ae6d-11cf-96b8-444553540000" codebase="http://fpdownload.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=9,0,28,0" width="500" height="426" id="onlinePlayer"><param name="allowScriptAccess" value="always" /><param name="movie" value="http://www.slideboom.com/player/player.swf?id_resource=631622" /><param name="quality" value="high" /><param name="bgcolor" value="#ffffff" /><param name="flashVars" value="mode=2&idResource=1837&siteUrl=http://www.slideboom.com" /><param name="allowFullScreen" name="true" /><embed src="http://www.slideboom.com/player/player.swf?id_resource=631622" quality="high" bgcolor="#ffffff" width="500" height="426" name="onlinePlayer" allowScriptAccess="always" type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/go/getflashplayer" allowFullScreen="true" flashVars="mode=0&idResource=631622&siteUrl=http://www.slideboom.com" /></object>
{%endcomment%}


The following diagram depicts a common use case where there are multiple data centers connected across the WAN, and each is running a different type of web server.

![httpSessionSharing1.jpg](/attachment_files/httpSessionSharing1.jpg)

The XAP Global HTTP Session Sharing architecture allows users to deploy their web application across these multiple data centers where the session is shared in real-time and in a transparent manner. The HTTP session is also backed by a data grid cluster within each data center for fault tolerance and high-availability.

With this solution, there is no need to deploy a database to store the session, so you avoid the use of expensive database replication across multiple sites. Setting up XAP for session sharing between each site is simple and does not involve any code changes to the application.

XAPs Global HTTP Session Management is designed to deliver maximum performance for the application with ZERO application code changes.

{%vbar title=Global HTTP Session Management features:%}
- **Reduce App/Web server memory footprint** storing the session within a remote JVM.

- **No code changes required** to share the session with other remote Web/App servers - Support **Serialized and Non-Serialized** Session attributes. Your attributes do not need to implement Serializable or Externalizable interface.

- **Transparent Session sharing** between any App/Web server - Any JEE app/web server (WebSphere , Weblogic , JBoss , Tomcat , Jetty , GlassFish...) may share their HTTP session with each other.

- **Application elasticity** - Support **session replication** across different App/Web applications located within the same or different data-centers/clouds allowing the application to scale dynamically without any downtime.

- **Unlimited number of sessions and concurrent users** support - Sub-millisecond session data access by using XAP In-Memory-Data-Grid.

- **Session replication over the WAN** support - Utilizing XAP Multi-Site Replication over the WAN technology.

- HTTP Session **data access scalability** - Session data can utilize any of the supported In-Memory-Data-Grid topologies ; replicated , partitioned , with and without local cache.

- **Transparent App/Web Failover** - Allow app server restart without any session data loss.

- Any session data type attribute support - **Primitive and Non-Primitive** (collections, user defined types) attributes supported.

- **Sticky session and Non-sticky** session support - Your requests can move across multiple instances of web application seamlessly.

- **Atomic HTTP request** session access support - multiple requests for the session attributes within the same HTTP request will be served without performing any remote calls. Master session copy will be updated when the HTTP request will be completed.

- **Delta update support** – Changes identified at the session attribute level. Minimal serialization overhead and network utilization between client and IMDG , between IMDG primary and backup instances and between remote IMDG clusters using WAN Gateway for WAN replication.

- **Compression support** - Session attributes may be compressed to support very large session storage with minimal performance impact.

- **Principle based session sharing** - Allows session sharing across different applications using SSO.


{%endvbar%}

# Application Session Sharing

Configure your web application to use the XAP session manager, deploy the XAP in-memory data grid (IMDG) and deploy your web application. That's it!

There is no need to change the web application or plug in any custom code in order to enable session sharing between servers running in remote data centers. In addition, you don't have to add the HTTP session classes to the IMDG classpath.

The below diagram shows a more detailed view of the IMDG deployment. In this case, there are multiple partitions for high scalability, as well as backup instances for redundancy. The WAN Gateway is also deployed and shows replication between remote site.

 ![httpSessionSharing2.jpg](/attachment_files/httpSessionSharing2.jpg)

The end-to-end path between the two data center nodes includes the servlet and GigaSpaces filters, and the IMDG with local cache and WAN Gateway.

 ![httpSessionSharing3.jpg](/attachment_files/httpSessionSharing3.jpg)

{% comment %}

### Load-Balancing Scenarios

The XAP Global HTTP Session Sharing support two Load-Balancing scenarios:

##### Session Sharing Scenario

{% section %}
{% column width=60% %}
Have `cacheManager.cacheSessionLocally = true` when you would like multiple web application instances to share the same HTTP session. In this case your load balancer should be configured to support *non-sticky sessions* routing requests to a different web application based on some load-balancing algorithm. This will improve the performance of your application by avoiding reading the latest copy of the session from the remote space on each page load.
{% endcolumn %}
{% column width=40% %}
 ![http-session-non-sticky.jpg](/attachment_files/http-session-non-sticky.jpg)
{% endcolumn %}
{% endsection %}

##### Session Failover Scenario

{% section %}
{% column width=60% %}
Have `cacheManager.cacheSessionLocally = false` when you would like the same web application instance to serve the same client and have the client to failover to another web application in case the original web application fails. In this case **sticky sessions** should be enabled at the HTTP load-balancer allowing the HTTP request associated with the same session to be routed always to the same Web container. When there is a web container failure, other Web container will take over and have the most up-to-date session retrieved from the In-Memory-Data-Grid.
{% endcolumn %}
{% column width=40% %}
 ![http-session-sticky.jpg](/attachment_files/http-session-sticky.jpg)
{% endcolumn %}
{% endsection %}

{% endcomment %}



# Configuration

### The Web Application Configuration

The web application requires a couple of configuration changes to the `web.xml` file in order to enabled XAP Session sharing:

{% highlight xml %}
<web-app>
		....
	<listener>
      		<listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>
    	</listener>
    	<listener>
      		<listener-class>com.gigaspaces.httpsession.sessions.GigaSpacesCacheManager</listener-class>
    	</listener>
        <filter>
        	<filter-name>GigaSpacesHttpSessionFilter</filter-name>
        	<filter-class>com.gigaspaces.httpsession.web.GigaSpacesHttpSessionFilter</filter-class>
        	<web:init-param>
        		<web:param-name>rewriteUrl</web:param-name>
            		<web:param-value>false</web:param-value> <!-- default true -->
        	</web:init-param>
        </filter>
        <filter-mapping>
            <filter-name>GigaSpacesHttpSessionFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>
</web-app>
{% endhighlight %}

{% note %}The **GigaSpacesHttpSessionFilter** must be the first filter defined.{% endnote %}

### Shiro configuration

shiro.ini settings

{: .table   .table-condensed   .table-bordered}
|Section|Property|Description|Required|Optional Values|Default Values|
|:------|:-------|:----------|:-------|:--------------|:-------------|
|main|connector| wrap SpaceProxy and perform operation aginst space|Yes|`com.gigaspaces.httpsession.SpaceConnector`|
|main|connector.url| Space url|Yes|`jini://*/*/<space_name>`|
|main|connector.username| Space username|No|`<space username>`|
|main|connector.password| Space password|No|`<space password>`|
|main|connector.sessionLease|Lease timeout in milliseconds|No|Any positive integer. Millisecond time unit| 1800000 |
|main|connector.readTimeout|Read timeout in milliseconds|No|Any positive interger. Millisecond time unit| 300000 |
|main|sessionManager|XAP Session Manager Implementation|Yes|com.gigaspaces.httpsession.GigaSpacesWebSessionManager|
|main|sessionManager.sessionDAO||Yes|$sessionDAO|
|main|sessionDAO|Provides a transparent caching layer between the components that use it and the underlying EIS (Enterprise Information System) session backing store |Yes|org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO|
|main|cacheManager|XAP extension of org.apache.shiro.cache.CacheManager Provides and maintains the lifecycles of `com.gigaspaces.httpsession.sessions.GigaSpacesCache` instances|Yes|com.gigaspaces.httpsession.sessions.GigaSpacesCacheManager|
|main|cacheManager.compressor|Set the compressor instance to be used|No|$compressor|`com.gigaspaces.httpsession.serialize.NonCompressCompressor`
|main|cacheManager.serializer|instance of the serializer implementation|Yes|$serializer|
|main|cacheManager.policy|instance of session policy implementation|Yes|$policy|
|main|cacheManager.connector|instance of space connector implementation|Yes|$connector|
|main|compressor|Provides compress functionality|No| Provides your own `com.gigaspaces.httpsession.serialize.Compressor` implementation or use one of the out of the box option:<br> 1.`com.gigaspaces.httpsession.serialize.CompressorImpl`<br>2.`com.gigaspaces.httpsession.serialize.NonCompressCompressor`|
|main|storeMode|Provide functionality of how to save changes to the space. there is tow sessions store mode full and delta.|Yes| use on of two options:<br> 1.`com.gigaspaces.httpsession.sessions.FullStoreMode` 2.`com.gigaspaces.httpsession.sessions.DeltaStoreMode`|
|main|storeMode.connector| Space connector to be used|Yes|$connector|
|main|storeMode.listener|Provides changes notification functionality. it must extends `com.gigaspaces.httpsession.policies.GigaspacesNotifyListener`|No| `listener`|
|main|storeMode.changeStrategy|define strategy of comparison and conflict detection response.|Yes|DeltaStoreMode:   1.`com.gigaspaces.httpsession.policies.LWWChangeStrategy` 2.`com.gigaspaces.httpsession.policies.FailFastChangeStrategy` 3.`com.gigaspaces.httpsession.policies.PartialChangeStrategy`|FullStoreMode: `com.gigaspaces.httpsession.policies.FullModeStrategy`    DeltaStoreMode: `com.gigaspaces.httpsession.policies.LWWChangeStrategy`
|main|listener|Fully qualified class name implementing `com.gigaspaces.httpsession.policies.GigaspacesNotifyListener`|No|`com.gigaspaces.httpsession.policies.TraceListener`|
|main|serializer|Provides serialization functionality|Yes| use you own implementation of `com.gigaspaces.httpsession.serialize.Serializer` or one of the out of the box options: 	1.`com.gigaspaces.httpsession.serialize.KryoSerializerImpl` (recomended)	2.`com.gigaspaces.httpsession.serialize.XStreamSerializerImpl`|
|main|serializer.logLevel|internal kryo logging level|No| 1. `NONE = 6` disables all logging.<br> 2. `ERROR = 5` is for critical errors. The application may no longer work correctly.<br> 3. `WARN = 4` is for important warnings. The application will continue to work correctly.<br> 4.`INFO = 3` is for informative messages. Typically used for deployment.<br> 5. `DEBUG = 2` is for debug messages. This level is useful during development.<br> 6. `TRACE = 1` is for trace messages. A lot of information is logged, so this level is usually only needed when debugging a problem. | `LEVEL_INFO = 3`
|main|serializer.classes|comma separate list full qualified class names to be loaded at the initialization of the Kryo Serializer|No||
|main|policy|Provides functionality of session policy to apply e.g. with and without authentication|Yes| Options:<br>1.`com.gigaspaces.httpsession.policies.SessionPolicyWithLogin` for sharing session cross multiple aplplications<br>2.`com.gigaspaces.httpsession.policies.SessionPolicyWithoutLogin` for single application session store|
|main|policy.connector = $connector|instance of space connector implementation|Yes||
|main|policy.storeMode = $storeMode|instance of space storeMode implementation|Yes||


The `shiro.ini` file should to be placed within the `WEB-INF` folder. See below examples for the `shiro.ini` file:

{%accordion id=acc0%}


{%accord title=Single Application Session Sharing Configuration Example... | parent=acc0%}

{% highlight console %}

	[main]
	# space proxy wraper
	connector = com.gigaspaces.httpsession.SpaceConnector
	connector.url = jini://*/*/sessionSpace
	# When using secured GigaSpace cluster, pass the credentials here
	# connector.username = <username>
	# connector.password = <password>
	# Default lease is 30 minutes - 30 * 60 * 1000 = 1800000
	connector.sessionLease = 1800000
	# Default read timeout is 5 minutes = 5 * 60 * 1000 = 300000
	connector.readTimeout = 300000

	sessionManager = com.gigaspaces.httpsession.GigaSpacesWebSessionManager

	#set the sessionManager to use an enterprise cache for backing storage:
	sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
	sessionManager.sessionDAO = $sessionDAO

	# ensure the securityManager uses our native SessionManager:
	securityManager.sessionManager = $sessionManager

	# whatever your CacheManager implementation is, for example:
	cacheManager = com.gigaspaces.httpsession.sessions.GigaSpacesCacheManager

	# Model Manager service
	storeMode = com.gigaspaces.httpsession.sessions.DeltaStoreMode
	storeMode.connector = $connector
	#storeMode.changeStrategy = com.gigaspaces.httpsession.policies.FailFastChangeStrategy
	listener1 = com.gigaspaces.httpsession.policies.TraceListener
	storeMode.listener = $listener1

	cacheManager.storeMode = $storeMode
	# Serialization Service
	serializer = com.gigaspaces.httpsession.serialize.KryoSerializerImpl
	serializer.logLevel = 1
	#### classes registation: class1, class2, ...,classN
	#serializer.classes = com.pak1.myClass1 , com.pak2.myClass2
	cacheManager.serializer = $serializer
	# Session Policy Service
	policy = com.gigaspaces.httpsession.policies.SessionPolicyWithoutLogin
	policy.connector = $connector
	policy.storeMode = $storeMode

	cacheManager.policy = $policy
	# space proxy setter
	cacheManager.connector= $connector

	# This will use GigaSpaces for _all_ of Shiro's caching needs (realms, etc), not just for Session storage.
	securityManager.cacheManager = $cacheManager

{% endhighlight %}
{%endaccord%}

{%accord title=Multiple Applications Session Sharing Configuration Example...  | parent=acc0%}
{% note %}Note that this example uses the basic authentication configuration but, Shiro has various authenticator types see [realm modules](http://shiro.apache.org/static/1.2.1/apidocs/org/apache/shiro/authc/class-use/AuthenticationException.html) {% endnote %}
{% highlight console %}

	[main]
	authc.loginUrl = /login.jsp

	# space proxy wraper
	connector = com.gigaspaces.httpsession.SpaceConnector
	connector.url = jini://*/*/sessionSpace
	# When using secured GigaSpace cluster, pass the credentials here
	# connector.username = <username>
	# connector.password = <password>
	# Default lease is 30 minutes - 30 * 60 * 1000 = 1800000
	connector.sessionLease = 1800000

	#sessionManager = org.apache.shiro.web.session.mgt.StandardWebSessionManager
	#sessionManager = org.apache.shiro.web.session.mgt.DefaultWebSessionManager
	sessionManager = com.gigaspaces.httpsession.GigaSpacesWebSessionManager

	#set the sessionManager to use an enterprise cache for backing storage:
	sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
	sessionManager.sessionDAO = $sessionDAO

	# ensure the securityManager uses our native SessionManager:
	securityManager.sessionManager = $sessionManager

	# whatever your CacheManager implementation is, for example:
	cacheManager = com.gigaspaces.httpsession.sessions.GigaSpacesCacheManager

	# Compression service
	#compressor = com.gigaspaces.httpsession.serialize.CompressorImpl
	#cacheManager.compressor = $compressor
	# Model Manager service
	#changeStrategy = com.gigaspaces.httpsession.policies.LastWriteWinsPolicy
	storeMode = com.gigaspaces.httpsession.models.DeltaStoreMode
	storeMode.connector = $connector
	#storeMode.changeStrategy = $changeStrategy

	cacheManager.storeMode = $storeMode
	# Serialization Service
	serializer = com.gigaspaces.httpsession.serialize.KryoSerializerImpl
	#serializer.classes = com.gigaspaces.httpsession.sessions.NestSerial , com.gigaspaces.httpsession.sessions.NestNonSerial
	cacheManager.serializer = $serializer
	# Session Policy Service
	policy = com.gigaspaces.httpsession.policies.SessionPolicyWithLogin
	policy.connector = $connector
	policy.storeMode = $storeMode

	cacheManager.policy = $policy
	# space proxy setter
	cacheManager.connector= $connector

	# This will use GigaSpaces for _all_ of Shiro's caching needs (realms, etc), not just for Session 	storage.
	securityManager.cacheManager = $cacheManager

	[users]
	## format: username = password, role1, role2, ..., roleN
	root = secret,admin
	guest = guest,guest
	presidentskroob = 12345,president
	darkhelmet = ludicrousspeed,darklord,schwartz
	lonestarr = vespa,goodguy,schwartz

	[roles]
	## format: roleName = permission1, permission2, ..., permissionN
	admin = *
	schwartz = lightsaber:*
	goodguy = winnebago:drive:eagle5

	[urls]
	## The /login.jsp is not restricted to authenticated users (otherwise no one could log in!), but
	## the 'authc' filter must still be specified for it so it can process that url's
	## login submissions. It is 'smart' enough to allow those requests through as specified by the
	## shiro.loginUrl above.
	/login.jsp = authc
	/** = authc
	##/logout = logout
	##/account/** = authc
	/remoting/** = authc, roles[b2bClient], perms["remote:invoke:lan,wan"
{% endhighlight %}
{%endaccord%}
{%endaccordion%}

<br>

### Web Application Libraries

The web application should include the following libraries within its `\WEB-INF\lib` folder:

* gs-session-manager-{{site.latest_maven_version}}.jar
* gs-runtime.jar

{% note %}
The `gs-runtime.jar` should be replaced with the relevant XAP `gs-runtime.jar` matching your environment.
{% endnote %}

# Deployment

The XAP IMDG should be deployed using one of the [topologies.](/product_overview/space-topologies.html) You may also include a reference to a WAN Gateway.

{% highlight bash %}
# To deploy the IMDG called `sessionSpace` start the XAP agent using:
<XAP-HOME>/bin/gs-agent

# and run the following command to deploy the session Space:
<XAP-HOME>/bin/gs deploy-space sessionSpace

{% endhighlight %}


{% tip %}See the [deploy-space]({%latestadmurl%}/deploy-command-line-interface.html) command for details.{% endtip %}

### Deploying the WAN Gateway

The [WAN Gateway]({%latestjavaurl%}/multi-site-replication-over-the-wan.html) should be deployed using your preferred replication topography, such as multi-master or master-slave. See the [WAN Replication Gateway](/sbp/wan-replication-gateway.html) best practice for an example of how a multi-master Gateway architecture can be deployed.

### Securing the XAP IMDG

When using a [Secure XAP cluster]({%latestjavaurl%}/securing-your-data.html) you can pass security credentials using following parameters in the `shiro.ini` file:

{%highlight console%}
connector.username = user
connector.password = pass
{%endhighlight%}


# Example

### Single-Site Deployment

The example can be deployed into any web server (Tomcat, JBoss, Websphere, Weblogic, Jetty, GlassFish)

1. Download the demo web application [demo app](/download_files/global-http-session-sharing.war).
2. Deploy a space named **sessionSpace**. You many have a single instance Space or deploy a clustered Space using the command line or GS-UI/Web-UI.
3. Deploy the `demo-app.war` into Tomcat (or any other app server).
4. Start your browser and access the web application via the following URL: http://localhost:8080/demo-app

{% note %}
The URL above assumes the Web Server configured to use port 8080.
{% endnote %}

{%panel%}

![httpSessionSharing4.jpg](/attachment_files/httpSessionSharing4.jpg)

1. Set some attributes with their name and value and click the **Update Session** button.
2. View the session updated within the space via the GS-UI or Web-UI.
3. Restart your web application and refresh the page. The session will be reloaded from the data grid.

{%endpanel%}


### Multi-Web Servers Deployment

You may share the HTTP session between different web servers. To test this on your local environment you can install multiple web servers, deploy the web application and have your browser access the same application via the same browser. See the below example:

{%panel%}
{%section%}
{%column width=50% %}
![httpSessionSharing8.jpg](/attachment_files/httpSessionSharing8.jpg)
{%endcolumn%}
{%column width=50% %}
![httpSessionSharing9.jpg](/attachment_files/httpSessionSharing9.jpg)
{%endcolumn%}
{%endsection%}



Hit the Refresh button when switching between the tabs. The session data will be refreshed with the relevant app server reading it from the Space.

{%endpanel%}

{% note %}
When deploying the web application WAR file please make sure the web app context will be identical.
{% endnote %}


# Apache Load-Balancer

### Configuration

Another option would be to use a load-balancer such as the [apache httpd](http://httpd.apache.org) and configure it to load-balance the web requests between the different web servers. Here is a simple setup:

{%accordion id=acc1 %}
{%accord parent=acc1 | title=Step 1:  Install Apache httpd%}
Install [apache httpd](http://httpd.apache.org).
{%endaccord%}

{%accord parent=acc1 | title=Step 2:  Create HttpSession.conf %}
Create a file named `HttpSession.conf` located at <Apache HTTPD 2.2 root>\conf\gigaspaces
{%endaccord%}

{%accord parent=acc1 | title=Step 3: Configure HttpSession.conf %}

Place the following within the `HttpSession.conf` file. The `BalancerMember` should be mapped to different URLs of your web servers instances. With the example below we have Tomcat using port 8080 and Websphere using port 9080.

{%highlight xml%}
<VirtualHost *:8888>
  ProxyPass / balancer://HttpSession_cluster/
  ProxyPassReverse / balancer://HttpSession_cluster/

  <Proxy balancer://HttpSession_cluster>
     BalancerMember http://127.0.0.1:8080 route=HttpSession_1
     BalancerMember http://127.0.0.1:9080 route=HttpSession_2
  </Proxy>
</VirtualHost>
{%endhighlight%}

{% note %} The `127.0.0.1` IP should be replaced with IP addresses of the machine(s)/port(s) of WebSphere/Tomcat instances.{% endnote %}
{%endaccord%}

{%accord parent=acc1 | title=Step 4:  Configure httpd.conf %}
 Configure the `<Apache2.2 HTTPD root>\conf\httpd.conf` to have the following:

{%highlight xml%}
Include "/tools/Apache2.2/conf/gigaspaces/*.conf"

LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule status_module modules/mod_status.so

Listen 127.0.0.1:8888

ProxyPass /balancer !
<Location /balancer-manager>
	SetHandler balancer-manager

	Order deny,allow
	Deny from all
	Allow from 127.0.0.1
</Location>
{%endhighlight%}

{% note %}The `/tools/Apache2.2` folder name should be replaced with your correct Apache httpd location. \\ The `127.0.0.1` IP should be replaced with appropriate IP addresses of the machine that is running apache.{% endnote %}
{%endaccord%}

{%accord parent=acc1 | title=Step 5: Restart Apache%}


Once you have the space running, Websphere running, Tomcat running, and Apache httpd configured, restart the Apache http. On windows you can use its service.

![httpSessionSharing7.jpg](/attachment_files/httpSessionSharing7.jpg)
{%endaccord%}

{%accord parent=acc1 | title=Step 6: Apache httpd balancer console%}
Once you performed the above steps, access the following URL:
{%highlight console%}
http://127.0.0.1:8888/HttpSession
{%endhighlight%}
You should have the web application running. Any access to the web application will be routed between Websphere and Tomcat. You can check this by accessing the Apache httpd balancer console:
{%highlight console%}
http://127.0.0.1:8888/balancer-manager
{%endhighlight%}

![httpSessionSharing6.jpg](/attachment_files/httpSessionSharing6.jpg)

You can shutdown Websphere or Tomcat and later restart these. Your web application will not lose its session data.
{%endaccord%}
{%endaccordion%}

<br/>

# Multi-Site Deployment

When deploying the [multi-site example](/sbp/wan-replication-gateway.html) you should change the `shiro.ini` for each site to match the local site Space URL. For example, to connect to the DE space you should have the web application use a `shiro.ini` with the following:

{%highlight java%}
connector.url = jini://*/*/wanSpaceDE?groups=DE
{%endhighlight%}

To connect to the US space you should have the web application use a `shiro.ini` with the following:

{%highlight java%}
connector.url = jini://*/*/wanSpaceUS?groups=US
{%endhighlight%}

# Library dependencies

Developers should include the following dependencies in pom.xml file.

{%highlight xml%}

	<repositories>
		<repository>
			<id>org.openspaces</id>
			<name>OpenSpaces</name>
			<url>http://maven-repository.openspaces.org</url>
			<releases>
				<enabled>true</enabled>
				<updatePolicy>daily</updatePolicy>
				<checksumPolicy>warn</checksumPolicy>
			</releases>
			<snapshots>
				<enabled>true</enabled>
				<updatePolicy>always</updatePolicy>
				<checksumPolicy>warn</checksumPolicy>
			</snapshots>
		</repository>
	</repositories>

	<dependency>
			<groupId>com.gigaspaces.httpsession</groupId>
			<artifactId>gs-runtime</artifactId>
			<version>{{site.latest_maven_version}}</version>
	</dependency>

	<dependency>
			<groupId>com.gigaspaces.httpsession</groupId>
			<artifactId>gs-session-manager</artifactId>
			<version>{{site.latest_maven_version}}</version>
	</dependency>
{%endhighlight%}

# Considerations

## Web Application Context

Global HTTP session sharing works only when your application is deployed as a non-root context. It is relying on browser cookies for identifying user session, specifically `JSESSIONID` cookie. Cookies are generated at a context name per host level. This way all the links on the page are referring to the same cookie/user session.

## WebSphere Application Server HttpSessionIdReuse Custom Property

When using XAP Global HTTP session sharing with applications deployed into WebSphere Application Server, please enable the [HttpSessionIdReuse](http://pic.dhe.ibm.com/infocenter/wasinfo/v7r0/index.jsp?topic=%2Fcom.ibm.websphere.express.doc%2Finfo%2Fexp%2Fae%2Frprs_custom_properties.html) custom property. In a multi-JVM environment that is not configured for session persistence setting this property to true enables the session manager to use the same session information for all of a user's requests even if the Web applications that are handling these requests are governed by different JVMs.

## Transient Attribute

An attribute specified as *transient* would not be shared and its content will not be stored within the IMDG. Your code should be modified to have this as a regular attribute that can be serialized.
