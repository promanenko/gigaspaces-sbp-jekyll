---
layout: post
title:  HTTP Session Sharing Tutorial
categories: SBP
parent: data-access-patterns.html
weight: 405
---

{% mute %} This tutorial explains how to use Global HTTP Session Sharing with XAP.{% endmute %}

{% panel %}
{%section%}
{%column width=15% %}
**XAP Version**<br>
**Last Updated**<br>
**Reference**<br>
**Screencast**<br>
**Examples**
{%endcolumn%}
{%column  width=50% %}
10.0<br>
November 2014<br>
[HTTP Session Sharing]({%latestjavaurl%}/global-http-session-sharing-overview.html)<br>
{%msword /sbp/attachment_files/httpsession/GlobalHttpSessionSharing-Screencast2014.pptx%} Screencast<br>
{%download /sbp/attachment_files/httpsession/demo-app.war%} demo-app.war <br>
{%download /sbp/attachment_files/httpsession/demo-app2.war%} demo-app2.war
{%endcolumn%}
{%column  width=10% %}
{%align right%}
**Author**
{%endalign%}
{%endcolumn%}
{%column  width=25% %}
{%align center%}
Shay Hassidim <br>
Deputy CTO, GigaSpaces<br>
{%endalign%}
{%endcolumn%}
{%endsection%}
{% endpanel %}


{%summary%}{%endsummary%}

This tutorial will show you:

1.	Run Single-Applications Session Sharing – sharing the same session between different Tomcat instances. <br>
2.	Run Multiple-Applications Session Sharing – sharing the same session between different applications running in different Web servers - Tomcat and JBoss.


# Prior running the Tutorial

Make sure you have enough disk space to install:

- XAP 10 - 150MB
- Tomcat 7.0.23 - 80MB
- JBoss 7 - 200MB

{%comment%}

http://archive.apache.org/dist/tomcat/tomcat-7/v7.0.23/bin/apache-tomcat-7.0.23.zip
http://download.jboss.org/jbossas/7.1/jboss-as-7.1.1.Final/jboss-as-7.1.1.Final.zip

Download the demo app:
https://dl.dropboxusercontent.com/u/7390820/demo-app.war
https://dl.dropboxusercontent.com/u/7390820/demo-app2.war

Documentation:
http://docs.gigaspaces.com/xap100/global-http-session-sharing-overview.html

Deck:
https://dl.dropboxusercontent.com/u/7390820/GlobalHttpSessionSharing-Screencast2014.pptx

Screencast:
https://dl.dropboxusercontent.com/u/7390820/http-session-screencast-part1.wmv
https://dl.dropboxusercontent.com/u/7390820/http-session-screencast-part2.wmv
https://dl.dropboxusercontent.com/u/7390820/http-session-screencast-part3.wmv
{%endcomment%}


# Single Application Session Sharing

With this session sharing model, the web user interacting with multiple web servers through a load balancer where each web server running the same web application. In most cases you will have multiple instances of the same web app running in a cluster configuration.  The load balancer rout requests based on the sticky session configuration to the relevant web server instance.

![http-session](/sbp/attachment_files/httpsession/http-session1.png)

In this scenario the session is shared via its **ID**. Where there is a web server failure or when a new web server is added to the cluster the session **ID** is used to retrieve the session from the IMDG. Only the updated attributes (delta) are sent to the IMDG when the session is updated. This ensure best application performance and minimal network bandwidth usage.


### Demo Flow
With this demo we will simulate session sharing between different tomcat instances by starting tomcat , running the application, terminating tomcat and later restarting tomcat without losing application HTTP Session data.

-	{%download http://archive.apache.org/dist/tomcat/tomcat-7/v7.0.23/bin/apache-tomcat-7.0.23.zip%} Tomcat 7.0.23
-	Install Tomcat by unzipping it into `c:\` or `d:\`
-	Install XAP by unzipping it into `c:\` or `d:\`
-	Start Tomcat by running `\apache-tomcat-7.0.23\bin\startup.bat`
-	Move to the `\gigaspaces-xap-premium-10.0.1-ga\bin` folder and start GigaSpaces agent by running:
{%highlight console%}
gs-agent.bat
{%endhighlight%}
-	Deploy a space named **sessionSpace**. You may have a single instance Space or deploy a clustered Space using the command line , GS-UI or the Web-UI. Here is how you can do this via the CLI
{%highlight console%}
gs.bat deploy-space sessionSpace
{%endhighlight%}
-	Double check the **shiro.ini** within the `demo-app.war` file located under the `WEB-INF` folder includes the lookup service host name as part of the `connector.url` property. With the example below we are using a lookup service running locally - hence the `localhost` is used:
{%highlight console%}
connector.url=jini://localhost/*/sessionSpace
{%endhighlight%}
-	Deploy the `demo-app.war` into Tomcat by placing it into \apache-tomcat-7.0.23\webapps
-	Start your web browser and access the web application via the following URL:

{%highlight console%}
http://localhost:8080/demo-app
{%endhighlight%}

The URL above assumes the Tomcat is configured to use port 8080.

{%panel%}
![http-session](/sbp/attachment_files/httpsession/http-session2.png)
{%endpanel%}

-	Add some attributes using the interface provided
-	View the session updated within the space via the GS-UI or Web-UI

![http-session](/sbp/attachment_files/httpsession/http-session3.png)

-	Identify the Session ID

![http-session](/sbp/attachment_files/httpsession/http-session4.png)

-	Terminate Tomcat by terminating its process
-	Refresh the page (press the `F5` key) – you should get an error. There is no web server to serve the HTTP request.
-	Start Tomcat again using :

{%highlight console%}
\apache-tomcat-7.0.23\bin\startup.bat
{%endhighlight%}

-	Refresh the page on the browser (press the `F5` key).
-	The session will be reloaded from the data grid. See tomcat console for log messages.


# Multi-Applications Session Sharing
With this session sharing model , the web user interacting with multiple web servers through a load balancer where each running a **different web application**. It may be one big web application that has been broken down into different modules each running within a different web server potentially running on a different host. Each web application may access different web application instance during the life cycle of the application.

In this scenario the session user login – the **principle** , used to identify the session to be shared across the different applications. You may have also multiple instances of each web application running to scale its activity. All will be sharing the same HTTP session.

Any new web server that will be added dynamically will be able to share the session. When the HTTP session is updated only the updated session attributes (delta) is sent to the IMDG. When the session is shared between different web applications where each might have a different version of the session - a timestamp check is performed to ensure the recent session is passed between the apps and also to be stored within the IMDG. In such a case , HTTP session attributes are merged to ensure no data is lost when navigating between the different web application modules.

### Demo Flow
With this demo we will simulate session sharing between Tomcat and JBoss web servers instances by starting Tomcat and JBoss , running the application on both , updating the HTTP session in both – where the session will be shared implicitly , terminating tomcat and JBoss and later restarting these without losing application HTTP session data.

-	{%download http://download.jboss.org/jbossas/7.1/jboss-as-7.1.1.Final/jboss-as-7.1.1.Final.zip%} JBoss 7.
-	Install JBoss 7 by unzipping it into `c:\` or `d:\`

![http-session](/sbp/attachment_files/httpsession/http-session5.png)

-	Double check the **shiro.ini** within the `demo-app2.war` file located under the `WEB-INF` folder includes the lookup service host name as part of the `connector.url` property. With the example below we are using a lookup service running locally - hence the `localhost` is used:
{%highlight console%}
connector.url=jini://localhost/*/sessionSpace
{%endhighlight%}
-	Deploy `demo-app2.war` into Tomcat by placing it into `\apache-tomcat-7.0.23\webapps`
-	Deploy `demo-app2.war` into JBoss by placing it into `\jboss-as-7.1.1.Final\standalone\deployments`
-	Edit `\jboss-as-7.1.1.Final\standalone\configuration\standalone.xml`

To have:

{%highlight xml%}
<socket-binding name="http" port="8081"/>
{%endhighlight%}

-	Start JBoss by running :

{%highlight console%}
\jboss-as-7.1.1.Final\bin\standalone.bat
{%endhighlight%}


-	Start your web browser and access the web application running in Tomcat via the following URL:

{%highlight console%}
http://localhost:8080/demo-app2
{%endhighlight%}

-	Login into the application running on Tomcat using :
o	user : root
o	password : secret

{%panel%}
![http-session](/sbp/attachment_files/httpsession/http-session6.png)
{%endpanel%}

-	Add some attributes using the interface provided

{%panel%}
![http-session](/sbp/attachment_files/httpsession/http-session7.png)
{%endpanel%}

-	Start your browser and access the web application running in JBoss via the following URL:

{%highlight console%}
http://localhost:8081/demo-app2
{%endhighlight%}

-	Login into the application running on JBoss using :
o	user : `root`
o	password : `secret`

{%panel%}
![http-session](/sbp/attachment_files/httpsession/http-session8.png)
{%endpanel%}

-	You should see all attributes added into the session via Tomcat are shared with the application running on JBoss.

{%panel%}
![http-session](/sbp/attachment_files/httpsession/http-session9.png)
{%endpanel%}

-	Add / Modify some attributes

{%panel%}
![http-session](/sbp/attachment_files/httpsession/http-session10.png)
{%endpanel%}

-	Switch to Tomcat:

{%highlight console%}
http://localhost:8080/demo-app2
{%endhighlight%}

-	Refresh the page (press the `F5` key) – see all the attributes been updated.

{%panel%}
![http-session](/sbp/attachment_files/httpsession/http-session11.png)
{%endpanel%}

-	Terminate both Tomcat and JBoss by terminating their process
-	Refresh these by press the `F5` key :

{%highlight console%}
http://localhost:8080/demo-app2
http://localhost:8081/demo-app2

{%endhighlight%}

-	An error message will be displayed as both Tomcat and JBoss cannot serve the HTTP request.
-	Start Tomcat and JBoss
-	Refresh the pages served by Tomcat and JBoss by pressing the `F5` key
-	Login via user: `root`, password: `secret`
-	See session been fully recovered.

{%panel%}
{%section%}
{%column width=50% %}
![http-session](/sbp/attachment_files/httpsession/http-session12.png)
{%endcolumn%}
{%column width=50% %}
![http-session](/sbp/attachment_files/httpsession/http-session13.png)
{%endcolumn%}
{%endsection%}
{%endpanel%}
