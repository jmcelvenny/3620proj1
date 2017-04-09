# Beacon Infrastructure Deployment

## Initial Setup

After the CloudLab experiment has been deployed, the following information should be collected:
 * Location of home node (H_SSH)
 * Root passwords for each node (H_PASS, C1_PASS, C2_PASS, I1_PASS)

Before running the script, password-less SSH must be set up between the machine running the install script and the Beacon home node. 

### Password-less SSH Setup

```
ssh -t -p 22 root@ms0423.utah.cloudlab.us
// Enter root password
cd ~/.ssh
ssh-keygen -t rsa
// Press ENTER for each prompt, leaving each blank
cat id_rsa.pub >> authorized_keys
cat id_rsa.pub >> authorized_keys2
logout
cd ~/.ssh
scp root@ms0423.utah.cloudlab.us:~/.ssh/id_rsa* ~/.ssh/
// Enter root password
cat id_rsa.pub >> authorized_keys
cat id_rsa.pub >> authorized_keys2
```

Now, you may run the install script. This script performs the following tasks:
 * Sets up password-less SSH between every node on the cluster
 * Installs necessary packages:
 	* sshpass: Simplifies ssh and scp bash script access
 	* python3, mpi4py, pip: Python 3 development libraries and package manager
 	* openmpi: OpenMPI libraries
 	* torque: Torque job scheduler (also known as PBS)
 * Sets up the home node as a Torque server/scheduler, and the two compute nodes as clients. 

### Torque Install Script

```
#!/bin/bash
USER="root"
H_SSH="ms0423.utah.cloudlab.us"
H_PASS="beacon#cluster"
C1_PASS="beacon#cluster"
C2_PASS="beacon#cluster"
I1_PASS="beacon#cluster"

ssh -t -p 22 $USER@$H_SSH << EOF
sudo su
cd ~/.ssh

rm id_rsa
rm id_rsa.pub
echo | ssh-keygen -t rsa -P ''
cat id_rsa.pub >> authorized_keys
cat id_rsa.pub >> authorized_keys2
echo "StrictHostKeyChecking no" >> config
rm known_hosts*
chmod go-w ~

apt-get update
echo y | apt-get install python-mpi4py
echo y | apt-get install python-pip
echo y | apt-get install openmpi-bin openmpi-doc libopenmpi-dev
echo y | apt-get install sshpass

# Copy SSH keys
sshpass -p $C1_PASS scp ~/.ssh/id_rsa* compute-1:~/.ssh/
sshpass -p $C2_PASS scp ~/.ssh/id_rsa* compute-2:~/.ssh/
sshpass -p $I1_PASS scp ~/.ssh/id_rsa* io-1:~/.ssh/

sshpass -p $C1_PASS ssh compute-1
apt-get update
echo y | apt-get install sshpass
cd ~/.ssh
cat id_rsa.pub >> authorized_keys
cat id_rsa.pub >> authorized_keys2
echo "StrictHostKeyChecking no" >> config
rm known_hosts*
chmod go-w ~

sshpass -p $C2_PASS ssh compute-2
apt-get update
echo y | apt-get install sshpass
cd ~/.ssh
cat id_rsa.pub >> authorized_keys
cat id_rsa.pub >> authorized_keys2
echo "StrictHostKeyChecking no" >> config
rm known_hosts*
chmod go-w ~

sshpass -p $I1_PASS ssh io-1
apt-get update
echo y | apt-get install sshpass
cd ~/.ssh
cat id_rsa.pub >> authorized_keys
cat id_rsa.pub >> authorized_keys2
echo "StrictHostKeyChecking no" >> config
rm known_hosts*
chmod go-w ~

ssh home
echo "128.110.152.158	beacon.server" >> /etc/hosts
echo y | apt-get install torque-server torque-client torque-mom torque-pam
echo "beacon.server" > /var/spool/torque/server_name
/etc/init.d/torque-mom stop
/etc/init.d/torque-scheduler stop
/etc/init.d/torque-server stop
killall -s 9 pbs_server
echo y | pbs_server -t create
killall -s 9 pbs_server
/etc/init.d/torque-scheduler start
/etc/init.d/torque-server start
echo "beacon.server" > /etc/torque/server_name
echo "beacon.server" > /var/spool/torque/server_priv/acl_svr/acl_hosts
echo "root@beacon.server" > /var/spool/torque/server_priv/acl_svr/operators
echo "root@beacon.server" > /var/spool/torque/server_priv/acl_svr/managers
echo "compute-2 np=8" >> /var/spool/torque/server_priv/nodes
echo "compute-1 np=8" >> /var/spool/torque/server_priv/nodes

qmgr -c 'set server scheduling = true'
qmgr -c 'set server keep_completed = 300'
qmgr -c 'set server mom_job_sync = true'
qmgr -c 'create queue batch'
qmgr -c 'set queue batch queue_type = execution'
qmgr -c 'set queue batch started = true'
qmgr -c 'set queue batch enabled = true'
qmgr -c 'set queue batch resources_default.walltime = 1:00:00'
qmgr -c 'set server default_queue = batch'
qmgr -c 'set server submit_hosts = beacon'
qmgr -c 'set server allow_node_submit = true'
qmgr -c "set queue batch max_running = 8"
qmgr -c "set queue batch resources_max.ncpus = 8"
qmgr -c "set queue batch resources_min.ncpus = 1"
qmgr -c "set queue batch resources_max.nodes = 2"
qmgr -c "set queue batch resources_default.ncpus = 1"
qmgr -c "set queue batch resources_default.neednodes = 1:ppn=1"
qmgr -c "set queue batch resources_default.nodect = 1"
qmgr -c "set queue batch resources_default.nodes = 1"

ssh compute-1
echo "10.10.1.3	beacon.server" >> /etc/hosts
apt-get update
echo y | apt-get install python-mpi4py
echo y | apt-get install python-pip
echo y | apt-get install openmpi-bin openmpi-doc libopenmpi-dev
echo y | apt-get install torque-server torque-client torque-mom torque-pam
/etc/init.d/torque-mom stop
/etc/init.d/torque-scheduler stop
/etc/init.d/torque-server stop
echo "beacon.server" > /etc/torque/server_name
echo '\$pbsserver	beacon.server' > /var/spool/torque/mom_priv/config
echo '\$logevent	255' >> /var/spool/torque/mom_priv/config
/etc/init.d/torque-mom start
/etc/init.d/torque-server start

ssh compute-2
echo "10.10.1.3	beacon.server" >> /etc/hosts
apt-get update
echo y | apt-get install python-mpi4py
echo y | apt-get install python-pip
echo y | apt-get install openmpi-bin openmpi-doc libopenmpi-dev
echo y | apt-get install torque-server torque-client torque-mom torque-pam
/etc/init.d/torque-mom stop
/etc/init.d/torque-scheduler stop
/etc/init.d/torque-server stop
echo "beacon.server" > /etc/torque/server_name
echo '\$pbsserver	beacon.server' > /var/spool/torque/mom_priv/config
echo '\$logevent	255' >> /var/spool/torque/mom_priv/config
/etc/init.d/torque-mom start
/etc/init.d/torque-server start

ssh home
EOF
```
Once the script is finished running, you may need to SSH into each compute node and run the following commands to kick-start the Torque instance:
```
/etc/init.d/torque-mom stop
/etc/init.d/torque-server stop
killall -s 9 pbs_server
/etc/init.d/torque-mom start
/etc/init.d/torque-server start
```
To see if the server is running, run the following:
```
qnodes
qstat -q
```
These commands will print the status of the Torque nodes and the job queue itself.

## Testing and Validation

To test our implementation of the Beacon infrastructure, our group ran an OpenMPI script on a set of Google Trace data. The following documents the steps taken and the received results.

### Logging in and requesting resources
```
// beacon is a test user, as root is not allowed to submit jobs to Torque
ssh beacon@home

beacon@home:~$ qsub -I -lnodes=2                                                                
qsub: waiting for job 20.beacon.server to start                                                 
qsub: job 20.beacon.server ready 

beacon@compute-1:~$ 

```
### Running script
```
beacon@compute-1:~$ mpirun -np 16 asg4.py                                                                                                       
--------------------------------------------------------------------------                                                                      
[[60620,1],1]: A high-performance Open MPI point-to-point messaging module                                                                      
was unable to find any relevant network interfaces:                                                                                             
                                                                                                                                                
Module: OpenFabrics (openib)                                                                                                                    
  Host: compute-2.beacon2.pdc-edu-lab-pg0.utah.cloudlab.us                                                                                      
                                                                                                                                                
Another transport will be used instead, although this may result in                                                                             
lower performance.                                                                                                                              
--------------------------------------------------------------------------                                                                      
[compute-1.beacon2.pdc-edu-lab-pg0.utah.cloudlab.us:02221] 15 more processes have sent help message help-mpi-btl-base.txt / btl:no-nics         
[compute-1.beacon2.pdc-edu-lab-pg0.utah.cloudlab.us:02221] Set MCA parameter "orte_base_help_aggregate" to 0 to see all help / error messages   
Unique jobs: 672074                                                                                                                             
Top 20 jobs by run time                                                                                                                         
Rank: 01        ID: 6217291782  Time: 2491521082981     Status: KILL                                                                            
Rank: 02        ID: 4867841244  Time: 2488059970906     Status: KILL                                                                            
Rank: 03        ID: 5782542009  Time: 2463114365081     Status: KILL                                                                            
Rank: 04        ID: 6035592839  Time: 2421933921466     Status: FAIL                                                                            
Rank: 05        ID: 6182354041  Time: 2340306707627     Status: KILL                                                                            
Rank: 06        ID: 6182354078  Time: 2340306707617     Status: KILL                                                                            
Rank: 07        ID: 6182353820  Time: 2340304587870     Status: KILL                                                                            
Rank: 08        ID: 6182354160  Time: 2340302039296     Status: KILL                                                                            
Rank: 09        ID: 6182354117  Time: 2340302039286     Status: KILL                                                                            
Rank: 10        ID: 6182353894  Time: 2340302038692     Status: KILL                                                                            
Rank: 11        ID: 6182353971  Time: 2340300639012     Status: KILL                                                                            
Rank: 12        ID: 6182354009  Time: 2340300639004     Status: KILL                                                                            
Rank: 13        ID: 6182353936  Time: 2340300638994     Status: KILL                                                                            
Rank: 14        ID: 6182353704  Time: 2340299854097     Status: KILL                                                                            
Rank: 15        ID: 6182353777  Time: 2340299181750     Status: KILL                                                                            
Rank: 16        ID: 6182353639  Time: 2340298451442     Status: KILL                                                                            
Rank: 17        ID: 6182353667  Time: 2340298451053     Status: KILL                                                                            
Rank: 18        ID: 6182353613  Time: 2340297782400     Status: KILL                                                                            
Rank: 19        ID: 6182353860  Time: 2340297020638     Status: KILL                                                                            
Rank: 20        ID: 6182366148  Time: 2340297020143     Status: KILL    

```

