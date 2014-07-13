---
layout: post
title:  Export-Import Tool
categories: SBP
parent: solutions.html
weight: 50
---

{% summary %}XAP IMDG Exporting-Importing Data Tool{% endsummary %}

{% tip %}
 **Author**:Shay Hasidim, John Burke, Christos Erototcritou<br/>
 **Recently tested with GigaSpaces version**: XAP 9.7<br/>
 **Last Update:** Apr 2014<br/>
{% endtip %}

# Introduction

With this tool we will demonstrate how to export data from a space via serializing it to a file. We can then re-import the data back into the space. The tool executes distributed tasks in 'preprocess' mode, which reads the serialization files and returns a de-duplicated list of the classes only.

![xap-export-import.png](/attachment_files/import-export-tool.jpg)

# Getting started

### Download the Export/Import Example

You can download the example project from [here](/download_files/Export_Tool.zip) and unzip into an empty folder.


### Build and Running the Tool

##### Step 1: Setup XAP maven plugin

- Add maven into your `PATH` in case it is missing:

{% highlight console %}
set PATH=d:\tools\apache-maven-3.0.4\bin;%PATH%
{% endhighlight %}

- run XAP maven plug-in installer:

{% highlight console %}
D:\gigaspaces-xap-premium-9.7.0-ga\tools\maven>installmavenrep.bat
{% endhighlight %}


##### Step 2: Deploy a space and write some data

- modify `<gsVersion>` within the `Export_Tool\pom.xml` to include the right XAP release - below example having XAP 9.7 (9.7.0-10496-RELEASE) as the `<gsVersion>` value:

{% highlight xml %}
<properties>
        <gsVersion>9.7.0-10496-RELEASE</gsVersion>
        <springVersion>3.1.3.RELEASE</springVersion>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
{% endhighlight %}


##### Step 3: Build the project

{% highlight console %}
cd <project_root>
mvn clean install
{% endhighlight %}

##### Step 4: Deploy a space and write some data
If you do not have an already deployed space with data, you will need to deploy a new space and write some dummy data to it.
 
##### Step 5:	Run the tool to export the objects

{% highlight console %}
cd target
java -classpath D:\gigaspaces-xap-premium-9.7.0-ga\lib\required\*;export-1.0-SNAPSHOT.jar;AccountClass.jar com.gigaspaces.tools.importexport.SpaceDataImportExportMain -e -l 127.0.0.1 -s space
{% endhighlight %}

In this case we are using a simple Account POJO.

{%note%}
Make sure that classes of the POJOs are set in the classpath before running.
{%endnote%}


{% highlight console %}
2014-04-06 20:34:14,029  INFO [com.gigaspaces.common] - (tid-23) : found 1 classes
2014-04-06 20:34:14,043  INFO [com.gigaspaces.common] - (tid-23) : starting export thread for xap.poc.common.Account
2014-04-06 20:34:14,043  INFO [com.gigaspaces.common] - (tid-23) : (tid-386) : reading space class : xap.poc.common.Account
2014-04-06 20:34:14,043  INFO [com.gigaspaces.common] - (tid-23) : (tid-386) : space partition contains 1239 objects
2014-04-06 20:34:14,043  INFO [com.gigaspaces.common] - (tid-23) : (tid-386) : writing to file : /Users/kemi/Documents/GigaSpaces/XAP/gigaspaces-xap-premium-9.7.0-ga/bin/xap.poc.common.Account.1.ser.gz
2014-04-06 20:34:14,044  INFO [com.gigaspaces.common] - (tid-23) : (tid-386) : read 1239 objects from space partition
2014-04-06 20:34:14,044  INFO [com.gigaspaces.common] - (tid-23) : (tid-386) : export operation took 215 millis
2014-04-06 20:34:14,044  INFO [com.gigaspaces.common] - (tid-23) : finished writing 1 classes
{% endhighlight %}

For each exported space class data `< XAP root\bin >` will have a zip file with the class instances content.

##### Step 6:	Run the tool to import the objects back into a space<br/>

Once you restart the data grid you can reload your data back. This will reload the data from the zip files into the space:
{% highlight console %}
cd target
java -classpath D:\gigaspaces-xap-premium-9.7.0-ga\lib\required\*;export-1.0-SNAPSHOT.jar com.gigaspaces.tools.importexport.SpaceDataImportExportMain -e -l 127.0.0.1 -s space
{% endhighlight %}


{%note%}
A space read call for each class is executed before trying to perform any import.
{%endnote%}

## Options
The tool supports the following arguments:

{: .table .table-bordered}
|:-------------|:-------------|:-----|
| Argument| Name          	| Description |
| -e             | Export 			| Performs space class export | 
| -i             | Import    	  	| Performs space class import |
| -g             | Group		    | The names of lookup groups - comma separated |
| -l             | Locators		    | The names of lookup services hosts - comma separated |
| -s             | Space		    | The name of the space |
| -c             | Classes		    | The classes whose objects to import/export - comma separated|
| -b             | Batch		    | The batch size - default is 1000|
| -p             | Partitions	    | The partition(s) to restore - comma separated|
| -t             | Test			    | Performs a sanity check|
