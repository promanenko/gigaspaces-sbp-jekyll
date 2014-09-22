---
layout: post
title:  Managing XAP with init.d
categories: SBP
parent: solutions.html
weight: 800
---


{% summary page %}{% endsummary %}
{% tip %}
**Summary**: TODO  <br>
**Author**:Patrick May<br/>
**Recently tested with GigaSpaces version**: XAP 10.0<br/>
**Last Update:** Sept 2014<br/>
{% endtip %}




# Introduction
The standard for managing long running daemons and daemon-like processes on Unix
systems is via a script residing in the /etc/init.d directory. Typically these scripts provide
four capabilities:

- start<br>
- stop <br>
- restart<br>
- status

This basic set of features can manage a simple   XAP configuration. More complex configurations require correspondingly more functionality.

# An init.d Script Shell
A typical init.d script follows this pattern:

{%highlight bash%}
#!/bin/sh

# Script description
start(){
}

stop(){
}

status(]{
}

case "$1" in start)
    start
    ;;

    stop)
        stop
    ;;

    restart)
     stop
     start
    ;;

    status)
     status
    ;;
*)
echo "Usage: $0 {start|stop|restart|status}"
exit 1
esac

{%endhighlight%}

For XAP, this script will be named simply /etc/init.d/xap.

# Configuring the   XAP Infrastructure

Managing a simple   XAP distributed space requires a certain amount of configuration information:

- The location where Java is installed<br>
- The location of the GigaSpaces XAP installation directory<br>
- The lookup group, if used<br>
- The IP addresses of the machines on which to run lookup services<br>
- The IP addresses of the machines on which to run GSMs<br>
- The number of GSCs per machine<br>

Several of these options are set at the top of the script:<br>

{%highlight bash%}

# Start and stop XAP infrastructure

#!/bin/sh

if [ -z "$JSHOMEDIR" ]; then
  echo "Please set JSHOMEDIR."
exit 1

fi

export GS_HOME=${JSHOMEDIR}
export PATH=${JSHOMEDIR}/bin:${PATH}
export LOOKUPLOCATORS=“server1:4174,server2:4174”

${JSHOMEDIR}/bin/setenv.sh
{%endhighlight%}

#### Starting the   XAP Infrastructure

The  XAP Agent is the easiest way to deploy the infrastructure components:

{%highlight bash%}
start() {
${JSHOMEDIR}/bin/gs-agent.sh gsa.gsc 4 gsa.lus 1 gsa.gsm 1 & >/dev/null
}
{%endhighlight%}



The lookup service (LUS) and GSM should only be started on the machines specified in
the LOOKUPLOCATORS environment variable. This can be accomplished either by
having different scripts on each machine or by checking for the machine name in the
script and passing the appropriate values to gs-agent.sh.

Multiple agents can be started to manage different zones.

#### Stopping the   XAP Infrastructure

Shutting down the   XAP infrastructure components is not as straightforward
as starting them, at least in versions 9.x and 10.0. Improvements are planned for later
releases. In the meantime, it is necessary to use the Admin API to cleanly shutdown.
The code to shutdown all Processing Units for a zone, undeploy the GSCs, and stop the Agent is:
 

This should be packaged in a jar file so it can be called like this:

{%highlight bash%}
stop() {
java -jar zone-shutdown.jar ${ZONE_NAME}
}
{%endhighlight%}


# Monitoring the   XAP Infrastructure

As with stopping the   XAP infrastructure, getting the status currently
requires using the Admin API. The code to determine how many GSCs are available for
a particular zone is:

This should be packaged in a jar file so it can be called like this:

{%highlight bash%}
stop() {
java -jar zone-status.jar ${ZONE_NAME}
}
{%endhighlight%}


### The Full init.d/xap Script

The full script looks like this:

{%highlight bash%}
#!/bin/sh

# Start and stop the   XAP infrastructure

if [ -z "$JSHOMEDIR" ]; then
  echo "Please set JSHOMEDIR."
exit 1

fi

export GS_HOME=${JSHOMEDIR}
export PATH=${JSHOMEDIR}/bin:${PATH}
export LOOKUPLOCATORS=“server1:4174,server2:4174”

${JSHOMEDIR}/bin/setenv.sh

start()
{
    ${JSHOMEDIR}/bin/gs-agent.sh gsa.gsc 4 gsa.lus 1 gsa.gsm 1 & >/dev/null
}

stop()
{
    java -jar zone-shutdown.jar ${ZONE_NAME}
}

status()
{
    java -jar zone-status.jar ${ZONE_NAME}
}

case "$1" in start)
    start
    ;;
    stop)
     stop
    ;;
    restart)
     stop
     start
    ;;
    status)
     status
    ;;
*)

echo "Usage: $0 {start|stop|restart|status}"
exit 1
esac

{%endhighlight%}

# Deploying the Space
Once the   XAP infrastructure is running on all provisioned machines, the
space can be deployed. This can either be done as part of the /etc/init.d/xap
script or a separate script. In either case, two conditions must be met:

1. All infrastructure components across all machines must be provisioned

2. The space should be deployed once and only once

The second condition means that only one machine should call the script to deploy the
space. The first means that the deployment script must use the Admin API to confirm
that all components are available. The code to determine how many GSCs are
available for a particular zone is:


This should be packaged in a jar file so it can be called like this:

{%highlight bash%}
java -jar zone-monitor.jar ${ZONE_NAME} ${GSC_COUNT}
{%endhighlight%}

This call will only return when the specified number of GSCs are available.
 