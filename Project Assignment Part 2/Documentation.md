## Network Topology and Node Distribution for the CloudLab Profile

#### All nodes are on the Clemson cluster since CentOS was required in order to run a LustreFS server and client(s).

The difference between the setup for this assignment on CloudLab and Beacon, is that Beacon runs 48 compute nodes and 6 I/O nodes while the assignment limits our nodes to 2 compute nodes, 1 I/O node, and a home node, meaning a proper ratio could not be achieved when scaling down. Only one I/O node was used since it can act as a Metadata Target (MDT), Management Server (MGS), and an Object Storage Server (OSS). It is not necessary to separate MDT/MGS from OSS. [1] Initially, our team used some CentOS nodes and some Ubuntu nodes. The CentOS nodes ran a LustreFS server while the Ubuntu nodes ran Torque. Difficulty installing a Lustre client on Ubuntu lead us to switch the whole cluster to CentOS.

In the CloudLab profile, “a central hub with a point-to-point connection” to each of the nodes was created forming a Star topology. [2] The Star network was used for a multitude of reasons, one of which being its similarity to the Beacon system and another being that it is more practical for MPI. This will covered more below.

## Similarities between the CloudLab Profile and Beacon

As mentioned in the previous section, the CloudLab profile was created using a Star topology. Beacon is also, by nature of a HPC, a Star network. Since Native Mode defaults to having “all code [running] directly on the coprocessors,” the focus will be placed in Offload Mode. In Offload Mode “code starts running on [the] host” module/node and allows for “parallel regions of code can be manually specified to run on the coprocessors using pragmas/directives.” [1] Given that the regions of code can be manually specified to run on the coprocessors and since the Beacon system also has use of the MPI library, a user on the Beacon system can specify a portion of the nodes to run the parallelizable code rather than sending the directive to all. “Every node [in a Star network] is connected to central node called hub or switch,” which is the case with Beacon and the CloudLab profile. [2] So, despite their not being a way to achieve a proper smaller scale ratio of nodes, both systems are using the same style network in order to achieve the same end goals.

## Similarities between the CloudLab Profile and Beacon

As mentioned in the previous section, the CloudLab profile was created using a Star topology. Beacon is also, by nature of a HPC, a Star network. Since Native Mode defaults to having “all code [running] directly on the coprocessors,” the focus will be placed in Offload Mode. In Offload Mode “code starts running on [the] host” module/node and allows for “parallel regions of code can be manually specified to run on the coprocessors using pragmas/directives.” [1] Given that the regions of code can be manually specified to run on the coprocessors and since the Beacon system also has use of the MPI library, a user on the Beacon system can specify a portion of the nodes to run the parallelizable code rather than sending the directive to all. “Every node [in a Star network] is connected to central node called hub or switch,” which is the case with Beacon and the CloudLab profile. [2] So, despite their not being a way to achieve a proper smaller scale ratio of nodes, both systems are using the same style network in order to achieve the same end goals.

## How the Scripts Deploy the Software Infrastructures
Before we could run the scripts to install torque and openMPI, our group had to set up a lustre server. This was run on the IO node of our system. After turning off the firewall and creating the lustre partitions, our script downloaded the necessary files needed to install lustre. The common way to install lustre is by using yum. However, this caused problems for our installation. Our script took care of this by turning off yum for the kernel and lustre before manually installing lustre using the downloaded packages. Our script then installed the lustre modules and other files needed. After local TCP connections were allowed, our script allowed ldev to use both object storage targets and test metadata targets. The lustre services needed were then set to start on boot and the system was rebooted. Finally, lustre was configured and the server was started. 
After setting up our IO node, we set up a client script to run on the compute nodes (compute-1, compute-2) and our home node. After turning off the firewall we made a directory for the lustre RPMS and downloaded the RPM files. Yum was used to install the lustre kernel and then lustre. After configuring lustre, we rebooted and mounted the scratch partition.
In order to run our next script we set up password-less ssh. 
After getting this script running, we ran another torque install script. The main points of this script were set up passwordless ssh between all of the nodes, install the necessary packages including sshpass, python, openmpi and torque.
After installing the necessary packages and setting up the passwordless ssh we connected to the nodes to get torque running. 
After completion of these scripts, our system was consistent with the beacon specifications.

## How We Validated Our System
The first thing we did to make sure our system was running was to run commands that listed both the status of the nodes and the job queue. 
In order to test the capabilities of the machine and actually test if it was working, we ran a script that sorted through Google Trace data in order to find the top 20 longest jobs runtimes. These results were consistent with what we found after running the same scripts on another established cluster computing system.
These results were analyzed to show that we have a fully functional cluster with a running job scheduler and parallel file system. 



[1]Beacon System Overview  https://www.nics.tennessee.edu/computing-resources/beacon/configuration  
[2] Star Network Topology
http://www.conceptdraw.com/How-To-Guide/star-network-topology
