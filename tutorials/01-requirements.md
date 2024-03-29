# Requirements

To move along this tutorial and set up the required tools and applications, you will
need access to a cloud computing infrastructure including a set of Virtual Machines (VMs).


<!-- You should have already received an email containing above information. -->

Before you begin, ensure that you have access to the EECS4222 lab in Azure Lab Services via this [link](https://labs.azure.com/virtualmachines). If you don't, follow the steps outlined in this [link](https://github.com/hamzehkhazaei/Azure_Lab) to complete the prerequisites.

So far, you should be able to get connected to the Windows machine by using RDP installed on your laptop. Also, you should have Hyper-V Manager installed on Windows. Make sure you have access to these resources and are comfortable using the `bash` to interact with the server before you continue this tutorial.

<!-- For this tutorial,
imagine the provided ssh private and public key is stored in `~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`
and thus will automatically be used for ssh communications; in case you stored the keys in a
different location, you need to use the `-i` option for all ssh commands, like below:

`# ssh username@ip_address -i private_key` -->

To set up a Kubernetes cluster, we will need different VMs to take `master`, `worker`, and `cluster head` roles.
The `master` nodes will be responsible for keeping the cluster running and scheduling resources on available
nodes while `worker` nodes will be responsible for running those workloads. `Cluster head` manages the nodes in the Kubernetes cluster and joins the worker nodes to the master.
Throughout this tutorial, we assume a cluster of following nodes: </br>

1. Cluster head: server1 with IP address 192.168.0.100
2. Worker node one: server2 with IP address 192.168.0.101
3. Worker node two: server3 with IP address 192.168.0.103
4. Master node: server4 with IP address 192.168.0.102

<strong>Note: </strong> You're not obligated to strictly follow the allocations shown at the top. Simply ensure that the VM with the highest resources is assigned to your Master node. In this case, the Master node has been allocated 121 GB of storage space.

<!-- ## OpenVPN Connection

To gain access to your VMs on our cloud, you will need to connect to our internal
network. To do so, you can use OpenVPN Connect to connect using the configuration file provided
to you. To use it, first download `OpenVPN Connect` from [their website](https://openvpn.net/download-open-vpn/). 
After the installation, you will be asked to create or import
a connection configuration. To do so, select the `File` tab and drag the provided connection
configuration onto OpenVPN Connect. Then, you will need to give the connection a name
and will be able to connect to the VPN server.

After connecting to the OpenVPN connection, you can test your connection by pinging the internal IP addresses
of the instances provided to you. -->

## SSH Access

To ssh onto the master or worker, use the `eecs` user. The password for 'eecs' user is 'eecs'. Open Command Prompt application (cmd.exe) on Windows for 4 times and connect to the server1, server2, server3, and server4 through ssh.

```sh
# ssh to the master
$ ssh eecs@192.168.0.102
```
```sh
# ssh to the worker one
$ ssh eecs@192.168.0.101
```
```sh
# ssh to the worker two
$ ssh eecs@192.168.0.103
```
```sh
# ssh to the cluster head
$ ssh eecs@192.168.0.100
```

<!-- 
To ssh onto the master or worker, use the default `ubuntu` user:

```sh
# ssh to the master
$ ssh ubuntu@10.1.1.1
# ssh to the worker
$ ssh ubuntu@10.1.1.2
# ssh to master if the private ssh key is not stored in the default place
$ ssh ubuntu@10.1.1.1 -i /PATH/TO/SSHKEY
``` -->

To learn more about SSH and how you can use it, watch [this tutorial](https://youtu.be/YS5Zh7KExvE).

<!-- ## Visual Studio Code

The easiest way to interact with your virtual machine is using Visual Studio Code 
Remote Development via SSH. To use this feature, you need to install the
[Visual Studio Code](https://code.visualstudio.com/) and the [Remote Development using SSH](https://code.visualstudio.com/docs/remote/ssh#_connect-to-a-remote-host)
extension. You can follow the VSCode tutorials to connect to the `master VM` to follow with the
tutorials.<br />
VS code -> View tab -> Command Palette <br />
Option: **Remote SSH: Connect to Host**<br />
Add new SSH Host:<br />
ssh eecs@192.168.0.100 <br/>
select a SSH configuration file to update -> /Users/username/.ssh/config<br />
from the bottom of the window select connect button<br />
(if asked enter the pass phrase for private key and press enter)<br />

We also recommend the installation of the following extensions (From the right panel select Extensions):

- `Python` by `Microsoft`
- `Jupyter` by `Microsoft`
- `Kubernetes` by `Microsoft`
- `Pylance` by `Microsoft` -->

## Initial Setup

First, we need to update all packages installed on all VMs. At the end of the following command, the OS asks for restarting some of the services, press `cancel` and move on.

```sh
# update the package list and upgrade installed packages on all machines
(master, workers and cluster head) $ sudo apt-get update && sudo apt-get upgrade -qy
```

In case you will be using more than one VM for your cluster, make sure that there
is network connectivity between your VMs by checking their `ping` status.

```sh
# ssh to the master and check connectivity with the workers
(master) $ ping 192.168.0.101
```

Make sure to replace `192.168.0.101` with your worker VM IP, if different.
You should see an output like this:

```console
PING 192.168.0.101 (192.168.0.101) 56(84) bytes of data.
64 bytes from 192.168.0.101: icmp_seq=1 ttl=64 time=0.708 ms
64 bytes from 192.168.0.101: icmp_seq=2 ttl=64 time=1.03 ms
64 bytes from 192.168.0.101: icmp_seq=3 ttl=64 time=0.608 ms
64 bytes from 192.168.0.101: icmp_seq=4 ttl=64 time=0.708 ms
^C
--- 192.168.0.101 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3019ms
rtt min/avg/max/mdev = 0.608/0.763/1.030/0.159 ms
```

## SSH-key

SSH keys are used to establish secure connections between two devices, allowing for secure and authenticated access to remote systems. SSH keys provide a more secure way to log in to a remote server than using a password alone. An SSH key pair typically consists of a private key, which should be kept confidential and stored on the client machine, and a public key, which can be shared and stored on remote servers. When connecting to a remote server using SSH, the client first authenticates using its private key, and the server then verifies the authenticity of the client's public key. If the public key matches the private key, the server grants access to the client. 
In this project, we generate ssh key on server1 and then copy the ssh public key to server2, server3, and server4.



```console
# Generating ssh key
$ ssh-keygen

Generating public/private rsa key pair.
Enter file in which to save the key (/home/eecs/.ssh/id_rsa): [enter]
Enter passphrase (empty for no passphrase): [enter]
Enter same passphrase again: [enter]
Your identification has been saved in /home/eecs/.ssh/id_rsa
Your public key has been saved in /home/eecs/.ssh/id_rsa.pub
...
```

```sh
# This copies the server1's public key to the server2, server3 and server4 authorized keys file
$ ssh-copy-id eecs@192.168.0.101
$ ssh-copy-id eecs@192.168.0.102
$ ssh-copy-id eecs@192.168.0.103
```

## The 'eecs' user on Ubuntu VMs
For some of the commands, you need to use `sudo` to run the command with the root access. `sudo` asks password from the user. By adding `eecs ALL=(ALL) NOPASSWD:ALL` at the end of the sudoers file, eecs user won't have to enter password each time using `sudo`. (Do the following part on all of the severs)

```sh
# open sudoers file
$ sudo visudo

# add following command at the end of the file
eecs ALL=(ALL) NOPASSWD:ALL

# press ctrl+o (write out), ENTER (save), and then ctrl+x (exit)
```
**Note:** The `eecs` has already have root access so to go to root mode you just need to enter `sudo sh`. 

<!-- 
```console
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=1.00 ms
64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=0.416 ms
64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=0.480 ms
64 bytes from 10.1.1.2: icmp_seq=4 ttl=64 time=0.471 ms
64 bytes from 10.1.1.2: icmp_seq=5 ttl=64 time=0.349 ms
64 bytes from 10.1.1.2: icmp_seq=6 ttl=64 time=0.348 ms
64 bytes from 10.1.1.2: icmp_seq=7 ttl=64 time=0.346 ms
64 bytes from 10.1.1.2: icmp_seq=8 ttl=64 time=0.362 ms
```
 -->
 
<!-- ## Firewall Configurations

The firewall configuration has been already done on your VMs, but generally we need the following
ports to be open for this tutorial:

- `TCP` port `6443` for Kubernetes API
- `UDP` port `8472` for Flannel VXLAN (Kubernetes CNI)
- `TCP` port `10250` for kubelet
- `TCP` port `80` for the web application
- `TCP` port `9090` for prometheus
- `TCP` port `8091` for locust
- `TCP` port `3000` for grafana -->

## Anaconda Installation

In this project, we will be using Python for interacting with our cluster. Using other
programming languages is also possible, but might need additional setup from you. You
can install [Anaconda](https://docs.conda.io/en/latest/) to act as the environment manager on your cluster.
**Run the following on your `master` node:**

```sh
# download miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
# install miniconda using non-interactive mode
bash ~/miniconda.sh -b -p $HOME/miniconda
# add bash hook for conda
echo 'eval "$(~/miniconda/bin/conda shell.bash hook)"' >> ~/.bashrc && source ~/.bashrc
```

After the installation is complete, you can test your installation using the following (It's not important to have the same version):

```console
$ conda --version
conda 23.11.0
```

## Jupyter Notebook Installation

Jupyter Notebook is a web-based interactive computational environment for creating and sharing documents that contain live code, equations, visualizations, and narrative text. Jupyter Notebook allows you to combine code, markdown, and multimedia in a single document, making it a powerful tool for data exploration and presentation. It is open-source software, built on top of the IPython (Interactive Python) shell, and runs on a variety of platforms, including Windows, MacOS, and Linux. You can install Jupyter on **master** VM using the conda package manager as follows:

```sh
$ conda install jupyter
```

<!-- Next, connect to the master node using visual studio code.
Then, open an empty file with `.py` extension to activate the python
extension on VS Code and click on the
`Select Python Interpreter` button on the bottom left corner of the window and
select `Python ... ('base': conda)` to use as the default python
environment. -->

Now that we have done our initialization step, you are ready to install
Kubernetes and other required tools on your cluster. 

[Next Step](02-kubernetes.md) -->
