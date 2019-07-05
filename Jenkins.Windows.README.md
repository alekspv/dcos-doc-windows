Jenkins on Mesos plugin (aka Jenkins Scheduler)
----------------
The `jenkins-mesos` plugin allows Jenkins to dynamically launch Jenkins slaves (both Linux and Windows) on a
Mesos cluster depending on the workload.

Put simply, whenever the Jenkins `Build Queue` starts getting bigger, this plugin
automatically spins up additional Jenkins slave(s) on Mesos so that jobs can be
immediately scheduled. Similarly, when a Jenkins slave is idle for a long time it
is automatically shut down.

The documentation describes how to configure Jenkins on Mesos plugin.

## How does Jenkins on Mesos work? ##
Jenkins on Mesos is designed to run workloads on dynamic slaves, 
which provided by Apache Mesos offers. Similar to Marathon, Jenkins on Mesos plays 
framework role - orchestrate workloads, however not require any Marathon involvment.

## Prerequisites: ##
You need to have access to a running DC/OS cluster. For instructions on setting up a DC/OS cluster, please refer to the
 [DC/OS docs website](https://docs.mesosphere.com).

## Installing the plugin ##

* Go to 'Manage Plugins' page in the Jenkins Web UI, you'll find the plugin in the 'Available' tab under the name 
'mesos'. Install `> 0.18.1` if you're looking Windows slaves. Otherwise only Linux slaves will work.

* Install the metrics plugin which is an optional dependency of this plugin, used for additional but not 
essential features.

### Configuring the plugin ###

Now go to 'Configure' page in Jenkins. If the plugin is successfully installed
you should see an option to 'Add a new cloud' at the bottom of the page. Add the
'Mesos Cloud' and give the path to the Mesos native library (e.g., libmesos.so on Linux or libmesos.dylib on OSX) 
(see the above section)
and the address (HOST:PORT) of a running Mesos master.

If you want to test immediately connectivity to Mesos, you can set 'On-demand framework registration' to 'no' 
and the framework will appear in Mesos as soon as you save. Otherwise it will register and unregister automatically when a build is scheduled on Mesos.


### Mesos slave setup ###

1) Ensure Mesos slaves have a `jenkins` user or the user the Jenkins master is running as. `jenkins`
 user should have JAVA_HOME environment variable setup.

2) DC/OS Agents require constraints setup so that Jenkins on Mesos will distinguish offered systems and spin 
up workloads on desired OS. 
 - DC/OS Linux agent constraints:  
`"os":"linux"`
To accomplish such add MESOS_ATTRIBUTES:
```
echo "MESOS_ATTRIBUTES=os:linux" >> /opt/mesosphere/etc/mesos-slave-common
```
Depends of agent type (Private or Public) restart the service: 
 `systemctl restart dcos-mesos-slave` or `systemctl restart dcos-mesos-slave-public`
 - DC/OS Windows agents need to be set with `"os":"windows"`. 
Mesos Agent on Windows started with help of the binary - `mesos-agent.exe`. By default `--attributes` flag is passed with the constraint on the stage of the agent join to the DC/OS cluster.



### Adding Slave Info ###

By default one 'Slave Info' will be created with default values for each field.
You can update the values/Add  more 'Slave Info'/Delete 'Slave Info' by clicking on 'Advanced'.
'Slave Info' can hold required information(Executor CPU, Executor Mem etc) for slave that need to be matched against Mesos offers.
Label name is the key between the job and the required slave to execute the job.
Ex: Heavy jobs can be assigned  label 'powerful_slave'(which has 'Slave Info' 20 Executor CPU, 10240M Executor Mem etc)
and light weight jobs can be assigned label 'light_weight_slave'(which has  'Slave Info' 1 Executor CPU, 128M Executor Mem etc).



### Mesos slave attributes ###

Mesos slaves can be tagged with attributes. This feature allows the Jenkins scheduler to pick specific
Mesos slaves based on attributes specified in JSON format. Ex. {"os":"windows"}

### Mesos authentication ###

By default the plugin (a Mesos framework) registers with Mesos master without authentication. To enable authentication:

  1. Click on the Add button next Set the `Framework principal` and `Framework Secret` fields in the plugin configuration page.

  2. Ensure the same credentials (`principal` and `secret`) are setup on the Mesos master via `"--credentials"` command line flag (See `./mesos-master.sh --help` for details).


### Checkpointing ###

Checkpointing can now be enabled by setting the "Checkpointing" option to yes in the cloud config. This will allow the Jenkins
master to finish running its slave jobs even if the Mesos slave process temporarily goes down. Note that Mesos slave(s) should
have checkpointing enabled for this to work. See [agent-recovery](http://mesos.apache.org/documentation/latest/agent-recovery/)
for more details.

### Configuring Jenkins jobs ###

Finally, just add the label name you have configured in Mesos cloud configuration -> Advanced -> Slave Info -> Label String (default is `mesos`) 
to the jobs (configure -> Restrict where this project can run checkbox) that you want to run on a specific slave type inside Mesos cluster.


### Docker containers ###

By default, the Jenkins slaves are run in the default Mesos container. To run the Jenkins slave inside a Docker container, there are two options.

	1) "Use Native Docker Containerizer" : Select this option if Mesos slave(s) are configured with "--containerizers=docker" (recommended).

	2) "Use External Containerizer" : Select this option if Mesos slave(s) are configured with "--containerizers=external".


### Docker Configuration ###

There are docker images recommended for Jenkins on Mesos plugin. See example of setup, in the table : 

| OS | Label   | Docker container (base image)  | Mesos Attributes (JSON format) | Specify JVM args | 
|----|----|----|----|----|
| Linux (CentOS, RHEL, Ubuntu) | mesos-linux | mesosphere/jenkins-dind:0.6.0-alpine | {"os":"linux"} |  -Xms512m -XX:+UseConcMarkSweepGC  -Djava.net.preferIPv4Stack=true | 
| Windows 1809 | mesos-windows | winamd64/openjdk:8-jre-windowsservercore-1809 | {"os":"windows"} | -Xms512m -XX:+UseConcMarkSweepGC |

> Note! java.net.preferIPv4Stack settings is missing at Open JDK 8 

#### Volumes ####

At a minimum, a container path must be entered to mount the volume. A host path can also be specified to bind mount the
 container path to the host path. This will allow persistence of data between slaves on the same node. The default 
 setting is read-write, but an option is provided for read-only use.

#### Parameters ####

Additional parameters are available for the `docker run` command, but there are too many and they change too often to
 list all separately. This section allows you to provide any parameter you want. Ensure that your Docker version on
  your Mesos slaves is compatible with the parameters you add and that the values are correctly formatted.
   Use the full-word parameter and not the shortcut version, as these may not work properly. Also, exclude
    the preceding double-dash on the parameter name. For example, enter `volumes-from` and `my_container_name` 
    to recieve the volumes from `my_container_name`. Of course `my_container_name` must already be on the Mesos slave
     where the Jenkins slave will run. This shouldn't cause problems in a homogenous environment where Jenkins slaves 
     only run on particular Mesos slaves.

### Over provisioning flags ###

By default, Jenkins spawns slaves conservatively. Say, if there are 2 builds in queue, it won't spawn 2 executors
 immediately. It will spawn one executor and wait for sometime for the first executor to be freed before deciding
  to spawn the second executor. Jenkins makes sure every executor it spawns is utilized to the maximum.
If you want to override this behaviour and spawn an executor for each build in queue immediately without waiting,
 you can use these flags during Jenkins startup:
`-Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85`

## Single-Use Slave ##

### Freestyle jobs ###

In the Build Environment settings, you may select "Mesos Single-Use Slave" to schedule disposal of the slave after 
the build finishes.

### Pipeline jobs ###

To schedule slave disposal from a Pipeline job:

    node('mylabel') {
        wrap([$class: 'MesosSingleUseSlave']) {
            // build actions
        }
    }
    
### Example of Jenkins Declarative pipeline job ###

Following jobs will scan version of OS :

Windows:

    stage "Verify Windows ver."
        node("mesos-windows") {
      	       bat "ver"
   	    }
    	    
Linux:

    stage("Verify Linux kernel ver.")
        node("mesos-linux") {
             sh "uname -a"
        }

> Note. It will work fully when [DCOS_OSS-5295](https://jira.mesosphere.com/browse/DCOS_OSS-5295) fixed. Otherwise, 
> existing Jenkins Mesos plugin will work for Linux slaves only. 