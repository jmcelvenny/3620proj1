## Network Topology and Node Distribution for the CloudLab Profile

#### All nodes are on the Clemson cluster since CentOS was required in order to run a LustreFS server and client(s).

The difference between the setup for this assignment on CloudLab and Beacon, is that Beacon runs 48 compute nodes and 6 I/O nodes while the assignment limits our nodes to 2 compute nodes, 1 I/O node, and a home node, meaning a proper ratio could not be achieved when scaling down. Only one I/O node was used since it can act as a Metadata Target (MDT), Management Server (MGS), and an Object Storage Server (OSS). It is not necessary to separate MDT/MGS from OSS. [1] Initially, our team used some CentOS nodes and some Ubuntu nodes. The CentOS nodes ran a LustreFS server while the Ubuntu nodes ran Torque. Difficulty installing a Lustre client on Ubuntu lead us to switch the whole cluster to CentOS.

In the CloudLab profile, “a central hub with a point-to-point connection” to each of the nodes was created forming a Star topology. [2] The Star network was used for a multitude of reasons, one of which being its similarity to the Beacon system and another being that it is more practical for MPI. This will covered more below.

## Similarities between the CloudLab Profile and Beacon

As mentioned in the previous section, the CloudLab profile was created using a Star topology. Beacon is also, by nature of a HPC, a Star network. Since Native Mode defaults to having “all code [running] directly on the coprocessors,” the focus will be placed in Offload Mode. In Offload Mode “code starts running on [the] host” module/node and allows for “parallel regions of code can be manually specified to run on the coprocessors using pragmas/directives.” [1] Given that the regions of code can be manually specified to run on the coprocessors and since the Beacon system also has use of the MPI library, a user on the Beacon system can specify a portion of the nodes to run the parallelizable code rather than sending the directive to all. “Every node [in a Star network] is connected to central node called hub or switch,” which is the case with Beacon and the CloudLab profile. [2] So, despite their not being a way to achieve a proper smaller scale ratio of nodes, both systems are using the same style network in order to achieve the same end goals.
##

- A section describing in details how the script(s) work to support the deployment of software infrastructures. If you customize existing work (i.e., the OpenStack default profile for CloudLab), makes sure that the original work is properly cited and that you describe what changes have been made. 



- A section describing validation results. 



[1]Beacon System Overview  https://www.nics.tennessee.edu/computing-resources/beacon/configuration
[2] Star Network Topology
http://www.conceptdraw.com/How-To-Guide/star-network-topology
