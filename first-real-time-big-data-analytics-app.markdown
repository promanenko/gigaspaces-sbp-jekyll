---
layout: post
title:  Your First Real Time Big Data Analytics Application
categories: SBP
weight: 1000
parent: examples.html
---

{% summary %}{% endsummary %}



{% section %}
{% column width=80% %}
{%wbr%}
We live almost every aspect of our lives in a real-time world. Think about our social communications; we update our friends online via social networks and micro-blogging, we text from our cell phones, or message from our laptops. But it's not just our social lives; we shop online whenever we want, we search the web for immediate answers to our questions, we trade stocks online, we pay our bills, and do our banking. All online and all in real time.
{% endcolumn %}
{% column width=20% %}
{%youtube ioHwEsARPWI  | Real Time Analytics%}
{% endcolumn %}
{% endsection %}

Real time doesn't just affect our personal lives. Enterprises and government agencies need real-time insights to be successful, whether they are investment firms that need fast access to market views and risk analysis, or retailers that need to adjust their online campaigns and recommendations to their customers. Even homeland security has come to increasingly rely on real-time monitoring.
The amount of data that flows in these systems is huge. Twitter, for example, 500 million Tweets per day, which is nearly 3,000 Tweets per second, on average.  At various peak moments through 2011, Twitter did as high as 8,000+ TPS, with at least one instance of over 25,000 tps. Facebook gets 100 billion hits per day with 3.2B Likes & Comments/day. Google get 2 billion searches a day. These numbers are growing as more and more users join the service.

This tutorial explains the challenges of a Real-time (RT) Analytics system using Twitter as an example, and show in details how these challenges can be met by using GigaSpaces XAP.


# The Challenge

Twitter users aren't just interested in reading tweets of the people they follow; they are also interested in finding new people and topics to follow based on popularity. This poses several challenges to the Twitter architecture due to the vast volume of tweets. In this example, we focus on the challenges relating to calculating the **word count** use case. The challenge here is straightforward:

1. Tens of thousands of tweets need to be stored and parsed every second.
1. Word counters need to be aggregated continuously. Since tweets are limited to 140 characters, we are dealing with hundreds of thousands of words per second.
1. Finally, the system needs to scale linearly so the stream can grow as the business grows.

These challenges are not simple to deal with as there are knock-on effects from the volume and analysis of the data, as follows:

- Tens of thousands of tweets to tokenize every second, meaning hundreds of thousands of words to filter -> **CPU bottleneck**
- Tens/hundreds of thousands of counters to update -> **Counters contention**
- Tens/hundreds of thousands of counters to persist -> **Database bottleneck**
- Tens of thousands of tweets to store in the database every second -> **Database bottleneck**

# Solution Architecture

{% section %}
{% column width=90% %}
In designing a solution, we need to consider the various challenges we must address.

The first challenge is providing **unlimited scalability** - therefore, we are talking about dynamically increasing resources to meet demand, and hence implementing a distributed solution using parallelized processing approach.

The second challenge is providing **low latency** - we can't afford to use a distributed file system such as Hadoop HDFS, a relational database or a distributed disk-based structured data store such as NoSQL database. All of these use physical I/O that becomes a bottleneck when dealing with massive writes. Furthermore, we want the business logic collocated with the data on a single platform for faster processing, with minimal network hops and integration issues.
{% endcolumn %}
{% column width=10% %}
{%popup /attachment_files/map_reduce.png %}
{% endcolumn %}
{% endsection %}

To overcome the latency challenge, we use an in-memory system of record. GigaSpaces XAP is built just for that. Its core component is [in-memory data grid](/product_overview/the-in-memory-data-grid.html) (IMDG, a.k.a. the Space) that partitions the data based on a specified attribute within the data object. The data grid uses a share nothing policy, and each primary node has consistent backup. In addition the grid keeps its SLA by self-healing crashed nodes, so it's completely consistent and highly-available.

The third challenge is the **efficient processing** of the data in a distributed system. To achieve this, we use the **Map** / **Reduce** algorithm for distributed computing on large data sets on clusters of computers. In the **Map** step, we normalize the data so we can create local counters. In the **Reduce** step, we aggregate the entire set of interim results into a single set of results.


{% section %}
{% column width=90% %}
In our Twitter example, we need to build a flow that provides the **Map** / **Reduce** flow in real time. For this we use XAP's Processing and Messaging features collocated with its corresponding data.

{% endcolumn %}
{% column width=10% %}
{%popup /attachment_files/map_reduce_tweets.png %}
{% endcolumn %}
{% endsection %}

Our solution therefore uses 2 modules for persisting and processing data, as follows:

- The feeder module persists raw tweets (the data) in the Space (IMDG)--The feeder partitions the Tweets using their ID (assumed to be globally unique) to achieve a scalable solution with rapid insertion of tweets into the Space.
- The processor module implements the **Map** / **Reduce** algorithm that processes tweets in the Space, resulting in real-time word counts. The tweets are then moved from the Space to the historical data store located in an Apache Cassandra Big-Data Store --The processor implements a workflow of events using the IMDG ability to store transient data in-memory and trigger processing activity in-memory.
The processor's **Map** phase has the following logical steps:

1. Tokenizes tweets into maps of tokens and writes them to the Space (triggered by the writing of raw tweets to the Space).
1. Filters unwanted words from the maps of tokens and writes the filtered maps to the Space (triggered by the writing of maps of tokens to the Space).
1. Generates a token counter per word, distributing the counters across the grid partitions for scalability and performance (triggered by the writing of filtered maps of tokens to the Space).

The processor's **Reduce** phase aggregates the local results into global word counters.


# Implementing the Solution

{% section %}
{% column width=90% %}
To implement our solution, we use Cassandra (or a local file) as the historical data tier and build a XAP application that processes and persists the data in real-time using the following modules:
{% endcolumn %}
{% column width=10% %}
{%popup  /attachment_files/twitter_topo.png %}
{% endcolumn %}
{% endsection %}

- The [`processor`](https://github.com/CloudifySource/cloudify-recipes/tree/master/apps/streaming-bigdata/processor) module is a XAP [processing unit]({%latestjavaurl%}/the-processing-unit-structure-and-configuration.html) that contains the Space and performs the real-time workflow of processing the incoming tweets. The processing of data objects is performed using event containers.

- The [`feeder`](https://github.com/CloudifySource/cloudify-recipes/tree/master/apps/streaming-bigdata/feeder) module is implemented as a processing unit thereby enabling the dynamic deployment of multiple instances of the feeder across multiple nodes, increasing the load it can manage, and thus the ability handle larger tweet streams. The processing unit contains the following feeder strategies:
    - The [`TwitterHomeTimelineFeederTask`](https://github.com/CloudifySource/cloudify-recipes/tree/master/apps/streaming-bigdata/feeder/src/main/java/org/openspaces/bigdata/feeder/TwitterHomeTimelineFeederTask.java) class, which feeds in tweets from Twitter's public timeline using [Spring Social](http://www.springsource.org/spring-social), converting them to a canonical [Space Document]({%latestjavaurl%}/document-api.html) representation, and writes them to the Space ,which in turn invokes the relevant event processors of the processor module.
    - The [`ListBasedFeederTask`](https://github.com/CloudifySource/cloudify-recipes/tree/master/apps/streaming-bigdata/feeder/src/main/java/org/openspaces/bigdata/feeder/ListBasedFeederTask.java) class is a simulation feeder for testing purposes, which simulates tweets locally, avoiding the need to connect to the Twitter API over the Internet.

- Optionally, the [`common`](https://github.com/CloudifySource/cloudify-recipes/tree/master/apps/streaming-bigdata/common) module for including items that are shared between the feeder and the processor modules (e.g. common interfaces, shared data model, etc.).

- The [`bigDataApp`](https://github.com/CloudifySource/cloudify-recipes/tree/master/apps/streaming-bigdata/bigDataApp) directory contains the recipes and other scripts required to automatically deploy, monitor and manage the entire application together with the [Cassandra](http://cassandra.apache.org/) back-end using [Cloudify](http://getcloudify.org).



# Building the Application

The following are step-by-step instructions building the application:

{%panel%}
1. Follow these [instructions]({%latestjavaurl%}/installation-guide.html#java-installation) to download and install the latest version of XAP.

2. Getting the Application
The application source can be found under `<XapInstallationRoot>/recipes/apps/streaming-bigdata folder`.
{%endpanel%}

Alternatively, you can download the source files in zip format from the [repository home on github](https://github.com/CloudifySource/cloudify-recipes/archive/2_5_1.zip)
The source are maintained in [Github Gigaspaces cloudify-recipes repository](https://github.com/CloudifySource/cloudify-recipes/tree/2_5_1/apps/streaming-bigdata).
If you're a github user and have already [setup the github client](http://help.github.com/mac-set-up-git/), you can [fork](http://help.github.com/fork-a-repo) the repository and clone it to your local machine, as follows:

{% highlight console %}
cd <project root directory>
git clone --branch=2_5_1 <your new repository URL>
{% endhighlight %}

We welcome your contributions and suggestions for improvements, and invite you to submit them by performing a [pull request](http://help.github.com/send-pull-requests/). We will review your recommendations and have relevant changes merged.

3. Install Maven and the GigaSpaces Maven plug-in
The application uses [Apache Maven](http://maven.apache.org/). If you don't have Apache Maven installed, please [download](http://maven.apache.org/download.html#Installation) and install it. Once installed:

- Set the `MVN_HOME` environment variable
- Add `$MVN_HOME/bin` to your path.
- Run the GigaSpaces Maven plug-in installer by calling the `<XapInstallationRoot>/tools/maven/installmavenrep.bat/sh` script.

4. Building the Application
Move to the `<applicationRoot>` folder (contains the application's project files).
Edit the pom.xml file and make sure the <gsVersion> include the correct   XAP release you have installed. For example if you have XAP {%version xap-version %} installed you should have the following:

{% highlight console %}
<properties>
	<gsVersion>{%version maven-version %}</gsVersion>
</properties>
{% endhighlight %}

To Build the project type the following at your command (Windows) or shell (*nix):

{% highlight console %}
mvn install
{% endhighlight %}

If you are getting **No gslicense.xml license file was found in current directory** error, please run the following:

{% highlight console %}
mvn package -DargLine="-Dcom.gs.home="<XapInstallationRoot>"
{% endhighlight %}

Where **XapInstallationRoot** should be XAP root folder - example:

{% highlight console %}
mvn package -DargLine="-Dcom.gs.home="c:/{%version gshome-directory %}"
{% endhighlight %}

The Maven build will download the required dependencies, compile the source files, run the unit tests, and build the required jar files. In our example, the following processing unit jar files are built:

- `<project root>/feeder/target/rt-feeder-XAP-9.x.jar`
- `<project root>/processor/target/rt-processor-XAP-9.x.jar`

Once the build is complete, a summary message similar to the following is displayed:

{% highlight console %}
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO]
[INFO] rt-analytics ...................................... SUCCESS [0.001s]
[INFO] rt-common ......................................... SUCCESS [2.196s]
[INFO] rt-processor ...................................... SUCCESS [11.301s]
[INFO] rt-feeder ......................................... SUCCESS [3.102s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 16.768s
[INFO] Finished at: Sun May 13 13:38:06 IDT 2012
[INFO] Final Memory: 14M/81M
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

# Running and Debugging the Application within an IDE

Since the application is a Maven project, you can load it using your Java IDE and thus automatically configure all module and classpath configurations.

- With [IntelliJ](http://www.intellij.com), simply click "File -> Open Project" and point to `<applicationRoot>/pom.xml`. IntelliJ will load the project and present the modules for you.
- With [Eclipse](http://www.eclipse.org), install the [M2Eclipse plugin](http://eclipse.org/m2e/m2e-downloads.html) and click "File -> Import" , "Maven -> Existing Maven Projects" , select the `streaming-bigdata` folder and click the `Finish` button.



{%section%}
{%column%}
{%popup /attachment_files/rt-ide1.jpg%}
{%endcolumn%}

{%column%}
{%popup /attachment_files/rt-ide2.jpg %}
{%endcolumn%}

{%column%}
{%popup /attachment_files/rt-ide3.jpg %}
{%endcolumn%}
{%endsection%}




Once the project is loaded in your IDE, you can run the application, as follows:

- In **Eclipse**, create two run configurations. One for the **feeder** and one for the **processor**. For both, the main class must be [`org.openspaces.pu.container.integrated.IntegratedProcessingUnitContainer`](http://www.gigaspaces.com/docs/JavaDoc{%currentversion%}/org/openspaces/pu/container/integrated/IntegratedProcessingUnitContainer.html).
Configure the GigaSpaces home folder using the **com.gs.home** system property:
`-Dcom.gs.home="c:\{%version gshome-directory %}"`
Configure the active spring profiles using the **spring.profiles.active** system property:
`-Dspring.profiles.active=list-feeder,file-archiver`

rt-processor project run configuration:

{%section%}
{%column%}

{%popup /attachment_files/rt-processor1.png%}
{%endcolumn%}

{%column%}

{%popup /attachment_files/rt-processor2.png%}
{%endcolumn%}
{%endsection%}

rt-feeder project run configuration:

{%section%}
{%column%}

{%popup /attachment_files/rt-feeder1.png%}
{%endcolumn%}

{%column%}

{%popup /attachment_files/rt-feeder2.png%}
{%endcolumn%}
{%endsection%}

- In IntelliJ, create two run configurations, with [`org.openspaces.pu.container.integrated.IntegratedProcessingUnitContainer`](http://www.gigaspaces.com/docs/JavaDoc{%currentversion%}/org/openspaces/pu/container/integrated/IntegratedProcessingUnitContainer.html) as the main class, and make sure that the feeder configuration uses the classpath of the `feeder` module, and that the processor configuration uses that of the `processor` module.

For more information about the `IntegratedProcessingUnitContainer` class (runs the processing units within your IDE), see [Running and Debugging Within Your IDE]({%latestjavaurl%}/running-and-debugging-within-your-ide.html).

{% tip %}
Make sure you have updated **gslicense.xml** located under the GigaSpaces XAP root folder with the license key provided as part of the email sent to you after downloading GigaSpaces XAP.
{% endtip %}

To run the application, run the **processor** configuration, and then the **feeder** configuration. An output similar to the following is displayed:

{% highlight console %}
2013-02-22 13:09:38,524  INFO [org.openspaces.bigdata.processor.TokenFilter] - filtering tweet 305016632265297920
2013-02-22 13:09:38,526  INFO [org.openspaces.bigdata.processor.FileArchiveOperationHandler] - Writing 1 object(s) to File
2013-02-22 13:09:38,534  INFO [org.openspaces.bigdata.processor.TweetArchiveFilter] - Archived tweet 305016632265297920
2013-02-22 13:09:38,535  INFO [org.openspaces.bigdata.processor.LocalTokenCounter] - local counting of a bulk of 1 tweets
2013-02-22 13:09:38,537  INFO [org.openspaces.bigdata.processor.LocalTokenCounter] - writing 12 TokenCounters across the cluster
2013-02-22 13:09:38,558  INFO [org.openspaces.bigdata.processor.GlobalTokenCounter] - Increment  local token arrive by 1
2013-02-22 13:09:38,606  INFO [org.openspaces.bigdata.processor.GlobalTokenCounter] - Increment  local token Reine by 1
2013-02-22 13:09:38,622  INFO [org.openspaces.bigdata.processor.GlobalTokenCounter] - Increment  local token pute by 1
2013-02-22 13:09:38,624  INFO [org.openspaces.bigdata.processor.GlobalTokenCounter] - Increment  local token lyc?e by 2
2013-02-22 13:09:41,432  INFO [org.openspaces.bigdata.processor.TweetParser] - parsing tweet SpaceDocument .....
2013-02-22 13:09:41,440  INFO [org.openspaces.bigdata.processor.TokenFilter] - filtering tweet 305016630734381057
2013-02-22 13:09:41,441  INFO [org.openspaces.bigdata.processor.FileArchiveOperationHandler] - Writing 1 object(s) to File
2013-02-22 13:09:41,447  INFO [org.openspaces.bigdata.processor.LocalTokenCounter] - local counting of a bulk of 1 tweets
2013-02-22 13:09:41,448  INFO [org.openspaces.bigdata.processor.LocalTokenCounter] - writing 11 TokenCounters across the cluster
2013-02-22 13:09:41,454  INFO [org.openspaces.bigdata.processor.TweetArchiveFilter] - Archived tweet 305016630734381057
2013-02-22 13:09:41,463  INFO [org.openspaces.bigdata.processor.GlobalTokenCounter] - Increment  local token Accounts by 1
2013-02-22 13:09:41,485  INFO [org.openspaces.bigdata.processor.GlobalTokenCounter] - Increment  local token job by 1
2013-02-22 13:09:41,487  INFO [org.openspaces.bigdata.processor.GlobalTokenCounter] - Increment  local token time by 1
{% endhighlight %}

## Switching between Online Feeder and the Test Feeder

You can switch between the On-Line `TwitterHomeTimelineFeederTask` Feeder and the Test `ListBasedFeederTask` Feeder. The former uses **real-time** Twitter time line data, while the latter uses simulated tweet data. By default, `ListBasedFeederTask` is enabled. To switch to `TwitterHomeTimelineFeederTask`

- add `-Dspring.profiles.active=twitter-feeder -Dtwitter.screenName=diggupdates` to the rt-feeder project VM arguments configuration.
- Get your twitter API secret keys:
   - Log in to http://dev.twitter.com/
   - Go to My applications and `Create a new Application`
   - Copy the `Consumer Key` and `Consumer Secret` to [feeder.properties](https://github.com/CloudifySource/cloudify-recipes/blob/master/apps/streaming-bigdata/feeder/src/main/resources/META-INF/spring/feeder.properties)
   - Click the `My Access Token` button on bottom of the page.
   - Copy the `Access Token` and `Access Token Secret` to [feeder.properties](https://github.com/CloudifySource/cloudify-recipes/blob/master/apps/streaming-bigdata/feeder/src/main/resources/META-INF/spring/feeder.properties)

{% warning %}
Since the [Twitter API uses rate limiting](https://dev.twitter.com/docs/rate-limiting), the twitter feeder is configured to poll once every 24 seconds (150 requests per hour) `-Dtwitter.delayInMs=24000`. Your feeder might stop if this rate limit is exceeded.
{% endwarning %}

# Running the Application with XAP Runtime Environment

The following are step-by-step instructions for running the application in XAP:

1. [Download](http://www.gigaspaces.com/LatestProductVersion) and [install]({%latestjavaurl%}/installation.html) XAP.
1. Edit `<XapInstallationRoot>/gslicense.xml>` and place the license key file provided with the email sent to you after downloading GigaSpaces XAP as the `<licensekey>` value.
1. Start a shell prompt in the `<XapInstallationRoot>/recipes/apps/streaming-bigdata` folder.
1. Run

{% highlight console %}
mvn package
{% endhighlight %}

to compile and package the source code into JARs

Step 1: Change to the `<XapInstallationRoot>/bin>` folder.
Step 2: Choose between the `twitter-feeder` and the `list-feeder` spring profile (All tweets are persisted to a file-archiver, cassandra-archiver is explained next):

{% inittab d10|top %}
{% tabcontent Unix twitter %}

{% highlight console %}
export GSC_JAVA_OPTIONS="-Dspring.profiles.active=twitter-feeder,file-archiver"
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Unix list %}

{% highlight console %}
export GSC_JAVA_OPTIONS="-Dspring.profiles.active=list-feeder,file-archiver"
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Windows twitter %}

{% highlight console %}
set GSC_JAVA_OPTIONS=-Dspring.profiles.active=twitter-feeder,file-archiver
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Windows list %}

{% highlight console %}
set GSC_JAVA_OPTIONS=-Dspring.profiles.active=list-feeder,file-archiver
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

Step 3: Start a [Grid Service Agent](/product_overview/service-grid.html#gsa) by running the `gs-agent.sh/bat` script. This will start two [GSCs](/product_overview/service-grid.html#gsc) (GSCs are the container JVMs for your processing units) and a [GSM](/product_overview/service-grid.html#gsm).

{% inittab d1|top %}
{% tabcontent Unix %}

{% highlight console %}
nohup ./gs-agent.sh >/dev/null 2>&1
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Windows %}

{% highlight console %}
start /min gs-agent.bat
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

Step 4: Then deploy the processor

{% inittab d1|top %}
{% tabcontent Unix %}

{% highlight console %}
./gs.sh deploy ../recipes/apps/streaming-bigdata/bigDataApp/processor/rt-analytics-processor.jar
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Windows %}

{% highlight console %}
gs deploy ..\recipes\apps\streaming-bigdata\bigDataApp\processor\rt-analytics-processor.jar
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

You should see the following output:

{% highlight console %}
Deploying [rt-analytics-processor.jar] with name [rt-processor-XAP-{% currentversion %}] under groups [{%version default-lookup-group %}] and locators []
Uploading [rt-analytics-processor] to [http://127.0.0.1:61765/]
Waiting indefinitely for [4] processing unit instances to be deployed...
[rt-analytics-processor] [1] deployed successfully on [127.0.0.1]
[rt-analytics-processor] [1] deployed successfully on [127.0.0.1]
[rt-analytics-processor] [2] deployed successfully on [127.0.0.1]
[rt-analytics-processor] [2] deployed successfully on [127.0.0.1]
Finished deploying [4] processing unit instances
{% endhighlight %}

Step 5: Next, deploy the feeder:

{% inittab d1|top %}
{% tabcontent Unix %}

{% highlight console %}
./gs.sh deploy ../recipes/apps/streaming-bigdata/bigDataApp/feeder/rt-analytics-feeder.jar
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Windows %}

{% highlight console %}
gs deploy ..\recipes\apps\streaming-bigdata\bigDataApp\feeder\rt-analytics-feeder.jar
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

{% warning %}
You will need XAP PREMIUM edition license key to deploy the processor in a clustered configuration
{% endwarning %}

You should see the following output:

{% highlight java %}
Deploying [rt-analytics-feeder.jar] with name [rt-analytics-feeder] under groups [{%version default-lookup-group %}] and locators []
Uploading [rt-analytics-feeder] to [http://127.0.0.1:61765/]
SLA Not Found in PU.  Using Default SLA.
Waiting indefinitely for [1] processing unit instances to be deployed...
[rt-analytics-feeder] [1] deployed successfully on [127.0.0.1]
Finished deploying [1] processing unit instances
{% endhighlight %}

Once the application is running, you can use the XAP UI tools to view your application , access the data and the counters and manage the application:

- For the Web Based UI run gs-webui.bat/sh and point your browser to localhost:8099 http://localhost:8099
- For the Rich Based UI run gs-ui.bat/sh

{% info title=More Deployment Options %}
To learn about additional options for deploying your XAP processing units, please see [Deploying onto the Service Grid]({%latestjavaurl%}/deploying-onto-the-service-grid.html)
{% endinfo %}

# Viewing Most Popular Words on Twitter

To view the most popular words on Twitter , start the GS-UI using the gs-ui.bat/sh , click the Query icon as demonstrated below and execute the following SQL Query by clicking the ![rt-tw6.jpg](/attachment_files/rt-tw6.jpg) button:

{% highlight console %}
select uid,* from org.openspaces.bigdata.common.counters.GlobalCounter order by counter DESC
{% endhighlight %}

You should see the top most popular words on twitter ordered by their popularity:

{% indent %}
{%popup /attachment_files/rt-tw4new.jpg %}
{% endindent %}

You can re-execute the query just by clicking the ![rt-tw5.jpg](/attachment_files/rt-tw5.jpg) button again. This will give you real-time view on the most popular words on Twitter.

# Persisting to Cassandra

Once raw tweets are processed, they are moved from the Space to the historical data backend store. By default, this points to a **simple flat file archiver** storage implemented with the `FileArchiveOperationHandler`. The example application also includes a Cassandra driver `CassandraArchiveHandler`.

{% tip %}
For more advanced persistency implementation see the [Cassandra Space Persistency Solution]({%latestjavaurl%}/cassandra-space-persistency.html).
{% endtip %}

{% tip %}
The next section uses cloudify to automate the manual steps described below
{% endtip %}

The following are step-by-step instructions configuring the application to persist to Cassandra:

1. Download, install, and start the Cassandra database. For more information, see Cassandra's [Getting Started](http://wiki.apache.org/cassandra/GettingStarted) page.
2. Define the TWITTER cassandra keyspace by running the following shell command:

{% highlight console %}
<cassandra home>/bin/cassandra-cli --host <cassandra host name> --file <project home>/bigDataApp/cassandra/cassandraKeyspace.txt
{% endhighlight %}

3. Deploy the processor and feeder
4. Start the Grid Components as described in the previous section. Do not deploy the application just yet. We need to start cassandra first.
We need to teardown the existing application since we injected the spring profile using environment variables that affect all Grid components.
Notice how this time we use the `cassandra-archiver` profile (instead of the `file-archiver profile`).

{% inittab d1|top %}
{% tabcontent Unix %}

{% highlight console %}
./gs.sh gsa shutdown
export GSC_JAVA_OPTIONS="-Dspring.profiles.active=twitter-feeder,cassandra-archiver -Dcassandra.hosts=127.0.0.1"
nohup ./gs-agent.sh >/dev/null 2>&1
./gs.sh deploy ../recipes/apps/streaming-bigdata/bigDataApp/processor/rt-analytics-processor.jar
./gs.sh deploy ../recipes/apps/streaming-bigdata/bigDataApp/feeder/rt-analytics-feeder.jar
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Windows %}

{% highlight console %}
gs.bat gsa shutdown
set GSC_JAVA_OPTIONS=-Dspring.profiles.active=twitter-feeder,cassandra-archiver -Dcassandra.hosts=127.0.0.1
start /min gs-agent.bat
gs deploy ..\recipes\apps\streaming-bigdata\bigDataApp\processor\rt-analytics-processor.jar
gs deploy ..\recipes\apps\streaming-bigdata\bigDataApp\feeder\rt-analytics-feeder.jar
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

5. You can view the data within Cassandra using the Tweet column family - Move to the Cassandra `bin` folder and run the `cassandra-cli` command:

{% highlight console %}
>cassandra-cli.bat
[default@unknown] connect localhost/9160;
[default@unknown] use TWITTER;
[default@TWITTER] list Tweet;
-------------------
RowKey: 0439486840025000
=> (column=Archived, value=00000000, timestamp=1361398666863002)
=> (column=CreatedAt, value=0000013cf9aea1c8, timestamp=1361398666863004)
=> (column=FromUserId, value=0000000039137bb7, timestamp=1361398666863003)
=> (column=Processed, value=01, timestamp=1361398666863001)
=> (column=Text, value=405f5f4c6f7665526562656c64652073656775696e646f2021205573613f20234172746875724d65754d61696, timestamp=136139866
6863000)
...
{% endhighlight %}

# Running the Example using Cloudify

To run the application with the Cassandra DB as one application on any cloud, we will use [Cloudify](http://getcloudify.org). A key concept with Cloudify is deploying and managing the entire application life cycle using a [Recipe](http://www.getcloudify.org/guide/{%latestcloudifyrelease%}/developing/recipes_overview.html). This approach provides total application life-cycle automation without any code or architecture change. Furthermore, it is cloud neutral so you don't get locked-in to a specific cloud vendor.

The following snippet shows the application's recipe:

{% highlight java %}
application {
	name="big_data_app"

	service {
		name = "feeder"
		dependsOn = ["processor"]
	}
	service {
		name = "processor"
		dependsOn = ["cassandra"]
	}
	service {
		name = "cassandra"
	}
}
{% endhighlight %}

The following snippet shows the life-cycle events described in the Cassandra service recipe:

{% highlight java %}
service {
  name "rt_cassandra"
  icon "Apache-cassandra-icon.png"
  numInstances 1
  type "NOSQL_DB"
  lifecycle{
		init 		"cassandra_install.groovy"
		preStart 	"cassandra_prestart.groovy"
		start 		"cassandra_start.groovy"
		postStart 	"cassandra_poststart.groovy"

	}
...
}
{% endhighlight %}

The following snippet shows the processing unit described in the processor recipe:

{% highlight java %}
service {
    name "processor"
    numInstances 4
    maxAllowedInstances 4
    statefulProcessingUnit {
        binaries "rt-analytics-processor.jar"
		springProfilesActive "cassandra-archiver,cassandra-discovery"
        sla {
            memoryCapacity 512
            maxMemoryCapacity 512
            highlyAvailable true
            memoryCapacityPerContainer 128
        }
    }
}
{% endhighlight %}

The application recipe is packaged, as follows:
![rt_app.png](/attachment_files/rt_app.png)

### Testing the application on a Local Cloud

XAP comes with a cloud emulator called `localcloud`. It allows you to test the recipe execution on your local machine. Follo these step-by-step instructions to installing and run the application on the `localcloud`:

1. Move to the <XapInstallationRoot>/tools/cli/ folder, and at the command (Windows) prompt, type: `cloudify.bat` (or at the shell *nix prompt, type: `cloudify.sh`).
1. To start the localcloud services, at the prompt, type `bootstrap-localcloud`. This may take few minutes.
1. To deploy the application, at the prompt, type:

{% highlight console %}
install-application <XapInstallationRoot>/recipes/apps/streaming-bigdata/bigDataApp
{% endhighlight %}

{% info title=Tracking installation progress %}
You can track the progress on the shell and on the web management console (localhost:8099).
{% endinfo %}

For more information, see [Deploying Applications](http://getcloudify.org/guide/2.6/deploying/deploying_apps.html) page.

### Running on Clouds

To run the application on one of the supported clouds, proceed the following steps:

1. Configure the cloud driver configuration file and get the cloud certificate. For more information, see [Post-Installation Configuration](http://getcloudify.org/guide/{%latestcloudifyrelease%}/setup/post_installation_configuration.html) page.
1. Bootstrap the cloud. For more information, see [The Bootstrapping Process](http://shlomo-tech-tav.github.com/guide/bootstrapping/bootstrapping_process) page.
1. To install and deploy the application, use the `install-application` command, as described in the previous section.

{% info title=Running XAP on the Cloud %}
In order to use your license on the cloud environment you should perform the following:

- cd to `<XAP installation root>/tools/cli/plugins/ecs/<cloud name>/upload`
- create directory cloudify-overrides
- copy your license(<XAP installation root>/gslicense.xml) to the above directory.
{% endinfo %}

{% comment %}
# The Design In Details

Now let's take a closer look at the components of the solution. Our solution is designed to efficiently cope with getting and processing the large volume of tweets. First, we partition the tweets so that we can process them in parallel, but we have to decide on how to partition them efficiently. Partitioning by user might not be sufficiently balanced, therefore we decided to partition by the tweet ID, which we assume to be globally unique. Then we need persist and process the data with low latency, and for this we store the tweets in memory.

This section describes the following components of the solution that implements these design decisions:
[Getting the Tweets](#get)
[Parsing the Tweets](#parse)
[Filtering the Tweets](#filter)
[Generating the Local Counters](#local)
[Generating Global Counters](#global)
[Persisting the Tweets to the Big Data Store](#persist)

{% anchor get %}

### Getting the Tweets

First, we need to get the tweets and store them in the Space (IMDG). In this example, we use [Spring Social](https://github.com/SpringSource/spring-social) to provide a Java interface to the Twitter API and get the tweets, and the [SpaceDocument]({%latestjavaurl%}/document-api.html) API of XAP to store the tweets. Using a SpaceDocument allows for a more flexible data model, the `SpaceDocument` being like a Map. The partitioning used the default 'ID' attribute.

The following snippet shows the relevant `TwitterHomeTimelineFeederTask` sections.

{% highlight java %}
public class TwitterHomeTimelineFeederTask implement Runnable {

    ...
    public SpaceDocument buildTweet(Tweet tweet) {
        return new SpaceDocument("Tweet", new DocumentProperties()
		.setProperty("Id", tweet.getId())
		.setProperty("Text", tweet.getText())
		.setProperty("CreatedAt", tweet.getCreatedAt())
		.setProperty("FromUserId", tweet.getFromUserId())
		.setProperty("ToUserId", tweet.getToUserId())
		.setProperty("Processed", Boolean.FALSE));
	}

    public void run() {
    	List<Tweet> userTimeline;
    	try {
    		log.info("Getting latest tweets from public timeline and feeding them into processing grid");
    		// Return all the tweets from the Twitter API
    		userTimeline = twitterTemplate.timelineOperations().getUserTimeline("BriefingcomSMU");
    	}
        catch(ApiException e){
        	log.log(Level.SEVERE, "Error getting tweets from public timeline from twitter", e);
        	return;
        }
    	try {
    		//according to the API we may get duplicate tweets if invoked with frequency of lower than 60 seconds.
    		//We will filter tweets which are duplicates
    		for (Tweet publicTweet : userTimeline) {
    			if (previousTimeLineTweets.contains(publicTweet.getId())){
    				continue;
    			}
    			logTweet(publicTweet);
    			gigaSpace.write(buildTweet(publicTweet));
    		}
    	} catch (DataAccessException e) {
    		log.log(Level.SEVERE, "error feeding tweets",e);
    	}
    	finally {
    		previousTimeLineTweets.clear();
    		for (Tweet publicTweet : userTimeline) {
    			previousTimeLineTweets.add(publicTweet.getId());
    		}
    	}
    }

    ...
}
{% endhighlight %}

{% anchor parse %}

### Parsing the Tweets

We have the raw data but we need to tokenize and filter it, and then update the local counters - these are the tasks performed by the **Map** phase of the **Map** / **Reduce** algorithm.

To generate this real-time flow, XAP uses the [event driven architecture of the event container]({%latestjavaurl%}/messaging-support.html). Specifically, we use a [Polling Container]({%latestjavaurl%}/polling-container.html) to listen for events relating to the writing of raw tweets to the Space. These events are configured using the `SQLQuery` returned by the `unprocessedTweet` method marked as `@EventTemplate`. Then, we tokenize & filter the tweet using the `@SpaceDataEvent` to mark the event handling method. The result is an object of type `TokenizedTweet` written to the Space.

The following snippet shows the relevant `TweetParser` sections.

{% highlight java %}
@EventDriven
@Polling(gigaSpace = "gigaSpace", concurrentConsumers = 2, maxConcurrentConsumers = 2)
@TransactionalEvent
public class TweetParser {
   ...
    /**
     * Event handler that receives a Tweet instance, processes its text and generates a listing of the tokens appearing in the
     * text and their respective count of appearance in the text, instantiates an instance of {@link TokenizedTweet} with this
     * data, and writes it to the space.
     *
     * @param tweet
     * @return {@link TokenizedTweet} containing a mapping of {token->count}
     */
    @SpaceDataEvent
    public SpaceDocument eventListener(SpaceDocument tweet) {
        log.info("parsing tweet " + tweet);

        Long id = (Long) tweet.getProperty("Id");
        String text = tweet.getProperty("Text");
        if (text != null) {
            gigaSpace.write(new TokenizedTweet(id, tokenize(text)));
        }

        tweet.setProperty("Processed", true);
        return tweet;
    }

    protected Map<String, Integer> tokenize(String text) {
        Map<String, Integer> tokenMap = newHashMap();
        StringTokenizer st = new StringTokenizer(text, "\"{}[]:;|<>?`'.,/~!@#$%^&*()_-+= \t\n\r\f
  ");

        while (st.hasMoreTokens()) {
            String token = st.nextToken();
            if (token.length() < MIN_TOKEN_LENGTH) {
                continue;
            }
            Integer count = tokenMap.containsKey(token) ? tokenMap.get(token) + 1 : 1;
            tokenMap.put(token, count);
        }
        return tokenMap;
    }
}
{% endhighlight %}

{% anchor filter %}

### Filtering the Tweets

The `TokenFilter` event handler is triggered by the writing of maps of tokens (`TokenizedTweet` objects) to the Space (marked as Non-Filtered). It's implemented as a batch polling container with a batch size of 100 entries. This class is responsible for filtering out a default list of irrelevant words like prepositions, and can be extended by applying additional values lists stored in the Space as "black lists". The filter updates the `TokenizedTweet` objects, removing the irrelevant words and writes them to the Space.

The following snippet shows the relevant `TokenFilter` sections.

{% highlight java %}
@EventDriven
@Polling(gigaSpace = "gigaSpace", concurrentConsumers = 2, maxConcurrentConsumers = 2, receiveTimeout = 5000)
@TransactionalEvent
public class TokenFilter {
    ...

    /**
     * Event handler that receives a {@link TokenizedTweet} and filters out non-informative tokens. Filtering is performed using
     * {@link #isTokenRequireFilter(String)}
     *
     * @param tokenizedTweet
     * @return the input tokenizedTweet after modifications
     */
    @SpaceDataEvent
    public TokenizedTweet eventListener(TokenizedTweet tokenizedTweet) {
        log.info("filtering tweet " + tokenizedTweet.getId());
        Map<String, Integer> tokenMap = newHashMap(tokenizedTweet.getTokenMap());
        int numTokensBefore = tokenMap.size();
        Iterator<Entry<String, Integer>> it = tokenMap.entrySet().iterator();
        while (it.hasNext()) {
            Entry<String, Integer> entry = it.next();
            if (isTokenRequireFilter(entry.getKey())) {
                it.remove();
            }
        }
        int numTokensAfter = tokenMap.size();
        tokenizedTweet.setTokenMap(tokenMap);
        tokenizedTweet.setFiltered(true);
        log.fine("filtered out " + (numTokensBefore - numTokensAfter) + " tokens from tweet " + tokenizedTweet.getId());
        return tokenizedTweet;
    }

    private boolean isTokenRequireFilter(final String token) {
        return filterTokensSet.contains(token);
    }

    private static final Set<String> filterTokensSet = newHashSet("aboard", "about", "above", "across", "after", "against",
            "along", "amid", "among", "anti", "around", "as", "at", "before", "behind", "below", "beneath", "beside", "besides",
            "between", "beyond", "but", "by", "concerning", "considering", "despite", "down", "during", "except", "excepting",
            "excluding", "following", "for", "from", "in", "inside", "into", "like", "minus", "near", "of", "off", "on", "onto",
            "opposite", "outside", "over", "past", "per", "plus", "regarding", "round", "save", "since", "than", "through", "to",
            "toward", "under", "underneath", "unlike", "until", "up", "upon", "versus", "via", "with", "within", "without");
}
{% endhighlight %}

{% anchor local %}

### Generating the Local Counters

This step completes the **Map** phase by taking the filtered maps of tokens, normalizing them, and counting the occurrences of relevant words per tweet. This is achieved using a PollingContainer named `LocalTokenCounter` that reads batches of filtered `TokenizedTweet` objects, and updates the counters which are `TokenCounter` objects in the Space. Note that `TokenCounter` objects are partitioned by the token for which they are aggregating the count.

The following snippet shows the relevant `LocalTokenCounter` sections.

{% highlight java %}
@EventDriven
@Polling(gigaSpace = "gigaSpace", passArrayAsIs = true, concurrentConsumers = 1, maxConcurrentConsumers = 1)
@TransactionalEvent
public class LocalTokenCounter {
    ...

    @SpaceDataEvent
    public void eventListener(TokenizedTweet[] tokenizedTweets) {
        log.info("local counting of a bulk of " + tokenizedTweets.length + " tweets");
        Map<String, Integer> tokenMap = newHashMap();
        for (TokenizedTweet tokenizedTweet : tokenizedTweets) {
            log.fine("--processing " + tokenizedTweet);
            for (Entry<String, Integer> entry : tokenizedTweet.getTokenMap().entrySet()) {
                String token = entry.getKey();
                Integer count = entry.getValue();
                int newCount = tokenMap.containsKey(token) ? tokenMap.get(token) + count : count;
                log.finest("put token " + token + " with count " + newCount);
                tokenMap.put(token, newCount);
            }
        }

        log.info("writing " + tokenMap.size() + " TokenCounters across the cluster");
        for (Entry<String, Integer> entry : tokenMap.entrySet()) {
            String token = entry.getKey();
            Integer count = entry.getValue();
            log.fine("writing new TokenCounter: token=" + token + ", count=" + count);
            clusteredGigaSpace.write(new TokenCounter(token, count), LEASE_TTL, WRITE_TIMEOUT, WriteModifiers.UPDATE_OR_WRITE);
        }
    }
}
{% endhighlight %}

{% anchor global %}

### Generating Global Counters

Now, the **Reduce** phase aggregates the local counters into global integer counters. This is achieved using another polling PollingContainer named `GlobalTokenCounter` listening for filtered `TokenCounter` objects. The container reads a batch of `TokenCounter` objects and updates the global count for each word. The global counter is an entry in the Space where the key is the token and the value is the aggregated count.

The following snippet shows the relevant `GlobalTokenCounter` sections.

{% highlight java %}
@EventDriven
@Polling(gigaSpace = "gigaSpace", concurrentConsumers = 1, maxConcurrentConsumers = 1)
@TransactionalEvent
public class GlobalTokenCounter {
    ...

    @SpaceDataEvent
    public void eventListener(TokenCounter counter,GigaSpace gigaSpace) {
    	log.info("Increment  local token " +counter.getToken() + " by " + counter.getCount());
    	IdQuery<GlobalCounter> counterIdQuery = new IdQuery<GlobalCounter>(GlobalCounter.class, counter.getToken());
    	ChangeResult<GlobalCounter> changeResult =
                          gigaSpace.change(counterIdQuery, new ChangeSet().increment("count", counter.getCount()));
    	//No counter for this token already exists
    	if (changeResult.getNumberOfChangedEntries() == 0)
    		gigaSpace.write(new GlobalCounter(counter.getToken(),counter.getCount()));

        if (log.isLoggable(Level.FINE)) {
            log.fine("+++ token=" + counter.getToken() + " count=" + counter.getCount());
        }
    }

}
{% endhighlight %}

{% anchor persist %}

### Persisting the Tweets to the Big Data Store

In this example, we use [Apache Cassandra](http://cassandra.apache.org/) as a historical Big Data store enabling future slicing and dicing of the raw tweets data. Similarly, we could use any database to persist the data, for example, another NoSQL data store or even to [Hadoop HDFS](http://hadoop.apache.org/hdfs/).

`TweetPersister` is a batch `PollingContainer` that uses the `ExternalPersistence` interface. `TweetPersister` writes batches of 100 parsed tweets to the NoSQL data store. In this case we use the `CassandraExternalPersistence` implementation that uses the [Hector Cassandra client for java](https://github.com/rantav/hector).

{% highlight java %}
public class CassandraExternalPersistence implements ExternalPersistence {
    ...
     @PostConstruct
    public void init() throws Exception {
        log.info(format("initializing connection to Cassandra DB: host=%s port=%d keyspace=%s column-family=%s\n" //
                , host, port, keyspaceName, columnFamily));
        cluster = getOrCreateCluster(keyspaceName, host + ":" + port);
        keyspace = createKeyspace(keyspaceName, cluster);
    }

    @Override
    public void write(Object data) {
        if (!(data instanceof SpaceDocument)) {
            log.log(Level.WARNING, "Received non document event");
            return;
        }
        SpaceDocument document = (SpaceDocument) data;
        Long id = document.getProperty("Id");
        log.info("persisting data with id=" + id);
        Mutator<String> mutator = createMutator(keyspace, stringSerializer);
        for (String key : document.getProperties().keySet()) {
            Object value = document.getProperty(key);
            if (value != null) {
                mutator.addInsertion(String.valueOf(id), //
                        columnFamily, //
                        createColumn(key, value.toString(), stringSerializer, stringSerializer));
            }
        }
        mutator.execute();
    }

    ...
}
{% endhighlight %}
{% endcomment %}

