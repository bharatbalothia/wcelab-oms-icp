# IBM Sterling OMS on IBM Cloud Private (ICP)

Installation and configuration of IBM Sterling OMS on IBM Cloud Private. Most is relevant to OMS on open source Kubernetes. 

## Cluster Configuration

Ensure that all default ports are open but are not in use. No firewall rules must block these ports. During installation the installer confirms that these ports are open.

For more information about the IBM Cloud Private default ports, see
https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.2/supported_system_config/required_ports.html

To check whether a port is open, run the following command:

``` shell
ssh -p <port_number> localhost
```

## Node preparation

### All Node preparation

** [NZ May2] Maybe we can create a script to prepare nodes (instead of step by step) **

INSTALLATION OF DOCKER

+ Download docker for ICP (icp-docker-17.09_x86_64.bin) from IBMs software catalogue
+ Make the file executable and ./icp-docker-17.09_x86_64.bin


Configure the /etc/hosts file on each node if no DNS is setup

``` shell
9.113.25.215    ciezc4b215.in.ibm.com
9.113.25.216    ciezc4b216.in.ibm.com
9.113.25.217    ciezc4b217.in.ibm.com
9.113.25.218    ciezc4b218.in.ibm.com
```

Install Python (version 2.6 to 2.9)

``` shell
apt-get install build-essential checkinstall
apt-get install libreadline-gplv2-dev libncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev
cd /usr/src
wget https://www.python.org/ftp/python/2.7.14/Python-2.7.14.tgz
tar xzf Python-2.7.14.tgz
cd Python-2.7.14
./configure --enable-optimizations
make altinstall
python2.7 -V
#python2.7 resides under /usr/local/bin
#create a symlink for python2.7 and link it to python.
#ln -s <source> <destination>
ln -s /usr/local/bin/python2.7 /usr/bin/python
python --version
```

Synchronize the clocks in each node in the cluster. To synchronize your clocks, you can use network time protocol (NTP).

``` shell
#ubuntu uses timesyncd instead of ntpd. For timesyncd use timedatectl command.
timedatectl list-timezones
timedatectl set-timezone Asia/Kolkata
date
timedatectl
#      Local time: Wed 2018-03-28 16:32:51 IST
#  Universal time: Wed 2018-03-28 11:02:51 UTC
#        RTC time: Wed 2018-03-28 11:02:51
#       Time zone: Asia/Kolkata (IST, +0530)
# Network time on: yes(means timesyncd is enabled)
#NTP synchronized: yes (time has been successfully synced)
# RTC in local TZ: no
#if timedatectl is disabled.. use below to enable
timedatectl set-ntp onn

```

Ensure that an SSH client is installed on each node

``` shell
apt-get install openssh-client aptitude ssh
```

### Master Node preparation

On master nodes, ensure that the vm.max_map_count setting is at least 262144 and the lower limit of the ephemeral port range is greater than 10240.

Run with sudo

``` shell
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
echo 'net.ipv4.ip_local_port_range="10240 60999"' >> /etc/sysctl.conf
sysctl -p
```

## INSTALLATION OF ICP

+ Login to boot node 
+ Download the installation files for the IBM cloud private i.e. download the ibm-cloud-private-x86_64-2.1.0.2.tar.gz file.
+ Extract the images and load them into Docker using the following command _[NZ May2] Why not use ```docker pull ibmcom/icp-inception```_

``` shell
tar xf ibm-cloud-private-x86_64-2.1.0.2.tar.gz -O | docker load
```

``` shell
mkdir /opt/ibm-cloud-private-2.1.0.2
cd /opt/ibm-cloud-private-2.1.0.2
### [NZ May2] Are we missing something here to mount the /opt/ibm-cloud-private-2.1.0.2 in host to /data in container?
docker run -v $(pwd):/data -e LICENSE=accept \
    ibmcom/icp-inception:2.1.0.2-ee \
    cp -r cluster /data
```

+ Create a secure connection from the boot node to all other nodes in cluster
  + Set up SSH in cluster
    + Log in to the boot node with an account with root access
    + Generate an SSH key and authorize it

    ``` shell
    ssh-keygen -b 4096 -f ~/.ssh/id_rsa -N ""
    ## Add new line in case previous key doesn't have new line [NZ May2] need to test that
    echo "" >> ~/.ssh/authorized_keys
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    ```
    + Add the key to a master, worker, proxy, management or Vulnerability Advisor (VA) node in the cluster.
      + From the boot node, add the SSH public key to the node.
      ``` shell
      ssh-copy-id -i ~/.ssh/id_rsa.pub <user>@<node_ip_address>
      ```
      + ssh in to the master, worker, proxy, management node to restart the SSH service
      ``` shell
      systemctl restart sshd
      ```
    + Add the IP address of each node in the cluster to the /<installation_directory> /cluster/hosts file.
      + Open the /<installation_directory>/cluster/hosts file.
      + Add the IP addresses for the different node types to the different sections of the file.
      
      ```
[master]
9.113.25.216

[worker]
9.113.25.217
9.113.25.218

[proxy]
9.113.25.215

[management]
9.113.25.215
      ```
      + If you use SSH keys to secure your cluster, in the /<installation_directory>/cluster folder, replace the ssh_key file with the private key file that is used to communicate with the other cluster nodes. Run this command:
      
      ``` shell
      cp ~/.ssh/id_rsa ./cluster/ssh_key
      ```
    
    + Move the image files for your cluster to the /<installation_directory>/cluster/images folder
    
    ```
    mkdir -p cluster/images
    mv /<path_to_installation_file>/ibm-cloud-private-x86_64-2.1.0.2.tar.gz  cluster/images/
    ```


