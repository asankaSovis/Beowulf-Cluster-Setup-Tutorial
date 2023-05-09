# Beowulf Cluster Setup

## 1. Introduction

A Beowulf cluster is a type of High-Performance Computing (HPC) cluster that is designed to perform parallel computations on large data sets or complex computational problems. It is named after the legendary warrior Beowulf, who is known for his strength and ability to overcome seemingly insurmountable challenges.

The main feature of a Beowulf cluster is that it is created from using all the open source resources available. Beowulf clusters are typically built using commodity hardware components: such as standard x86-based processors, Ethernet switches, and network interface cards. The nodes in a Beowulf cluster are connected through a high-speed local area network (LAN), which allows them to communicate and share resources efficiently.

One of the key features of a Beowulf cluster is its ability to distribute computing tasks across multiple nodes in parallel, allowing for faster processing times and improved performance. This is achieved through the use of a Message-Passing Interface (MPI), which allows nodes to communicate and coordinate their efforts in executing complex computations.

Beowulf clusters are commonly used in scientific research, engineering and other fields where large-scale computational tasks are required. They can be used to simulate complex physical processes, analyze massive data sets, and perform other computationally intensive tasks that would be impractical or impossible to perform on a single machine.

Overall, Beowulf clusters represent a powerful and cost-effective solution for high-performance computing, allowing organizations to harness the power of parallel computing to solve complex problems and drive innovation. Implementing a Beowulf cluster involves several steps, including hardware selection, software configuration, and network setup.

## 2. Hardware Selection

The hardware components required to follow this tutorial are as follows.

- More than one x86 based computers (These can be low powered)
- A router/switch with enough ethernet ports and cables to connect all the computers and internet access
- A monitor, keyboard and optionally a mouse
- USB Stick (At least 8GB)
- Separate computer for setting up the cluster

In order to make this tutorial the below components are used. It is not essential to use exactly these items but these are mentioned simply to get an idea of how to choose these components.

- Three 64-bit HP desktop computers with 4GB of RAM, CPUs with four cores at 1.5 GHz and a hard disk of 500GB
- ADSL router with four ethernet ports
- Dell monitor and logitech keyboard
- 8GB USB stick

## 3. Choosing an Operating System

In the very definition of a Beowulf cluster, it is mentioned **Open Source** software. Thus, it is essential to use an opensource operating system such as Linux to run in these computers. The most common OS used is [Ubuntu Server](https://ubuntu.com/download/server) although any of the available Linux OS would be suitable with slight variations of the software used and steps mentioned. In this example [Ubuntu Server 22.04.2](https://releases.ubuntu.com/22.04.2/ubuntu-22.04.2-live-server-amd64.iso) is used however, other versions should also work with the above disclaimers.

## 4. Preparing for the Setup

Before starting to implement the cluster, few things have to be taken care of. These are essential for the correct setting up of the cluster. Make sure to follow these steps carefully before moving further.

### 1. Setting up the Hardware

The first step is to setup the hardware correctly. This includes connecting the computers to the router via the ethernet cables. Pick one computer to be the master and the rest of the computers to be the slaves. The master computer or *master node* as refered to in Beowulf terms, is the computer that runs the program itself. This node breaks the process into multiple blocks and send to the slave computers or *nodes* to be processed. The router should be connected to the internet by suitable means. Internet connection is not essential for the operation of the Beowulf cluster but is only important for setting up the cluster.

The monitor, keyboard and optionally the mouse can be connected to the master node; however, note that these have to be removed and reconnected to the other computers as well.

### 2. Installing the Operating System

The best way to install the operating system on the hardware is to use a bootable USB stick. Use a separate computer and download the ISO of the installer of the preferred Linux OS, in the case of this tutorial, [Ubuntu Server 22.04.2](https://releases.ubuntu.com/22.04.2/ubuntu-22.04.2-live-server-amd64.iso) is used as previously mentioned.

Also download the [Rufus](https://rufus.ie/en/) bootable USB tool. In this example [Rufus Version 3.22 (Build 2009)](https://rufus.ie/downloads/) is used, however the version number has little to no effect on the setup. Other bootable USB creators can also be used.

Open Rufus and choose the prefered USB device under *Device*. Then click on *Boot Selection* and choose *Disk or ISO image (Please select)* and click on *SELECT* to choose the ISO file. Navigate to the ISO file and choose it. Other options should be left at defaults and click on *START*. For the warning, click *Ok* although note that this process formats the pen drive and all data currently in it will be lost. For the next page, keep the defaults and click on *Ok* as well. This will start the process of formatting the device into a bootable pen drive. Wait for this process to complete and remove the pen drive.

Next we should install the OS on the hardware. For this, connect the pen drive to each computer one after the other and install the OS. In certain computers, when booting the OS in EFI mode will result in installation errors. In cases like this, try switching to Legacy BIOS before continuing with the installation. The steps involved might change from computer to computer on how to switch to Legacy mode and how to enter boot menu. Please refer online for the specific models.

The installation steps are normal. In all of the steps, the defaults will work fine. In each page, highlight the option *Done*. It is not essential to setup the network as this will be done in a future step. Downloading additional packages is also not necessary. Following the steps in [this tutorial](https://ubuntu.com/server/docs/install/step-by-step) will be sufficient. In the **Identitiy** screen however, under *Your name:*, *Your servers name:* and *Pick a username:*; enter the following:

- '**master**' for the computer you wish to use as the main computer of the cluster
- '**node1**', '**node2**', '**node3**', etc. for the rest of the computers

It is not essential to follow this naming convention but will be helpful in identifying the compputers apart. For the *Choose your password:* and *Confirm your password:*, enter a common password for all computers. This will be helpful in keeping consistency although it's not essential to have the same password in all computers. Once the installation is complete, select *Reboot*. Every time the computers are turned on, enter the computer name and password to login to the computers.

## 5. Setting up the Beowulf Cluster

### 1. Introduction

As mentioned before, Beowulf cluster is a computer cluster made up of opensource components. Thus in order to set it up, the following components need to be setup in the computers.

- Message Passing Interface
- Network File System
- Hydra Process Manager

Make sure all the previous steps are followed correctly and that you have all of the computers ready with Ubuntu Server (Or any other Linux distro) installed.

### 2. Setting Up the IP addresses

In order to continue, each computer must have a unique static IP address. For example, in our case, we will use the IP address of the *master* as *192.168.1.5*, *node1* as *192.168.1.6*, and *node2* as *192.168.1.7*, etc. It is not mandatory to use these exact IP addresses but using a consistent IP address assigning might come in handy. Make sure you remember which computer has which IP address.

In order to setup static IP addresses, first make sure that all computers are correctly connected via ethernet to the router/switch. If this is confirmed, continue with the setup process. The below steps have to be performed on all the computers, thus disconnect and connect the keyboard and monitor to each computer and continue them.

1. Enter the command `ip a` on the terminal of each computer. This will list out all the network interfaces connected to the computer. Find the one connecting the computer to the router/switch. Usually this might be labelled as eno0, eno1, enp0s25 or eth0.

2. Once identified, move to the folder `/etc/netplan`. (Use the command `cd /etc/netplan`) Once in this folder, check if a file named `99_config.yaml` or simillar exist. (Use the command `ls`) If so open this using **Nano** or any other text editor. If it doesn't exist, create a file named `99_config.yaml`. (Use the command `sudo nano 99_config.yaml`) Enter the below information in the file.

```
network:
  version: 2
  renderer: networkd
  ethernets:
    eno0:
      addresses:
        - 192.168.1.5/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
          search: [mydomain, otherdomain]
          addresses: [10.10.10.1, 1.1.1.1]
```

Under `addresses:`, include the IP address to be assigned for that particular computer. In this example, we've included the IP address we provided for the *master*. Under `routes:`, next to `via:`, include the IP address of the router. This is typically **192.168.1.1**.

3. After complete, exit the text editor while saving the file. (*Ctrl + X -> Y -> Enter* for nano)

4. Use the command `sudo netplan apply` to apply the configuration.

5. On each computer, enter the command `ping 192.168.1.5` where the IP address is to be replaced by the IP addresses of other nodes. Try for each node to each of the other nodes connected. If the configuration is correctly performed, you will recieve a ping. If not, go through the steps and see if all of them are done correctly and that the network is setup correctly. A reason why this might fail is becuase the firewall refuses the connection. In that case, try disabling the firewall.

6. Next step is to add the hostnames to the `hosts` file. For this, move to the `/etc` folder. Edit the `hosts` file. Make sure the following hosts also exist in this file.

```
127.0.0.1       localhost
192.168.42.50   master
192.168.42.51   node1
192.168.42.52   node2
```

The IP addresses must match the IP address of the relevant node. Add all of the nodes you have here.

### 3. Creating the MPI User

A common user must exist across all computers to facilitate with the communication. For this reason, we will create a user called `mpiuser` across all the computers. This username as well as the UUID must be common to facilitate with the SSH authorization. Use the following command to create a user in all computers.

```
sudo adduser mpiuser --uid 999
```

If the UUID 999 is already taken, do the same command but with a different number. However make sure that the same UUID is provided across all computers for this user. Also make sure to include a number equal to or more than 999. Make sure to use a consistent password to avoid confusion. (Note that for certain commands, you might need to switch back to the main account; for example, commands that require `sudo` priviledges. Entering the command `exit` will switch you back to the main account.)

### 4. Setting up the Network File System (NFS)

Network File System (NFS) is a distributed file system protocol that allows a user on a client computer to access files over a network as if those files were on the local computer. With NFS, a user can browse the remote file system, read and write files, and even execute programs located on the remote server. This makes it possible for multiple users to share files and collaborate on projects from different locations.

For this, the computers require an internet connection. If not, connect each computer to the internet and continue the steps. It is ok to disconnect the computers from the router/switch if required. Complete the steps and reconnect to the router/switch.

Use `sudo apt-get update` and `sudo apt-get upgrade` to get the applist and apps upto date. Then install NFS as explained below.

For the *master* node, enter the below command.

```
sudo apt-get install nfs-kernel-server
```

For the rest of the nodes, use the below command.

```
sudo apt-get install nfs-common
```

This will install the relevant NFS package on each computer.

### 5. Sharing the Home folder

Across all the computers, the `/home/mpiuser` directory has to be shared via NFS. For this, the `/home/mpiuser` directory has to be owned by the *mpiuser*. However, this folder is already created while adding the user, which means that the *mpiuser* already has access to it. To verify this, the following command can be used.

```
master:~$ ls -l /home/ | grep mpiuser
```

If the folder is already owned by the mpiuser, this should return a message simillar to below.

```
drwxr-xr-x 7 mpiuser mpiuser 4096 May 30 05:55 mpiuser
```

If you want to share some other folder, use the below command on the *master* node with the specific folder you need to share.

```
sudo chown mpiuser:mpiuser /path/to/shared/dir
```

Now, the `/home/mpiuser` or any other folder created before of the *master* node needs to be shared with the rest of the nodes. For this, the relevant folder needs to be added to the ` /etc/exports` file. To do this, navigate to the `/etc` folder. (Use the command `cd /etc`) Now edit the file with the text editor (Use the command `sudo nano exports`) and add the following entry to it. This should only be done in the *master* node. 

```
/home/mpiuser *(rw,sync,no_subtree_check)
```

Once done, save the file, (*Ctrl + X -> Y -> Enter* for nano) and then restart the NFS kernel using the below command.

```
sudo service nfs-kernel-server restart
```

To check if the folder is shared correctly, use the following command on the master node.

```
showmount -e master
```

If correctly performed, this should print the folder `/home/mpiuser` or the relevanr folder on the terminal.

Next step is to check if its accessible with the other nodes. Before that, we need to make sure the firewall doesn't block the communications between computers. For this you can either disable the firewall or enter the following command on all computers. The IP address should be the gateway address. (*xxx.xxx.xxx.0/24)

```
sudo ufw allow from 192.168.1.0/24
```

Once complete, check the connectivity with the `ping` command used before.

Now, we can mount the shared folder on other nodes. For this, we use the below command on all nodes.

```
sudo mount master:/home/mpiuser /home/mpiuser
```

If this command hangs or fails, there's an issue with the configuration. If the command succeeds, test if the folder is shared across all nodes. For this, you can create a file in the `/home/mpiuser` directory of the *master* and see if it appears in the rest of the nodes. (Use the command `touch hello` to create a file)

If everything is correct so far, we have to setup the nodes to automatically mount this folder at boot. For this, move to the folder `/etc/`, (Use the command `cd /etc/`) then edit the file `fstab` on all computers. (Use the command `sudo nano fstab`) Add the below entry to this file.

```
master:/home/mpiuser /home/mpiuser nfs
```

Now reboot all computers and see if the system is correctly working, for example the `hello` file is accessible from all nodes.

### 6. Setting up the Secure Shell (SSH)

SSH (Secure Shell) is a cryptographic network protocol that provides a secure way to access remote systems over an unsecured network. It was developed as a replacement for Telnet and other insecure remote shell protocols. In order to continue, SSH has to be setup on all computers.

Connect each computer to the internet and use the following command to install SSH.

```
sudo apt-get install ssh
```

Once done on all computers, SSH keys need to be generated under the *mpiuser* for all computers. Since the home folder is shared across all computers, these keys will be accessible to the *master*. On each computer, switch to the *mpiuser*, (Use the command `su mpiuser`) and then enter the following command.

```
ssh-keygen
```

When asked for a passphrase, leave it empty (hence passwordless SSH). Now switch to the master node and enter the following command under *mpiuser*.

```
ssh-copy-id localhost
```

If done correctly, you will get a message that says someting simillar to '*Number of key (s) added:     3*'.

If configured correctly, now you might be able to SSH into any of the *nodes* from the *master*. Try to use the below command to enter into SSH from *master* to *node1*.

```
ssh node1
```

This should change the prompt from `mpiuser@master:~$ ` to `mpiuser@node1:~$ `. This means that all command executed will be executed inside of the *mpiuser* of the *node1*. For example, the command `echo $HOSTNAME`, will print `node1` instead of `master`. An advantage of this setup is that using the command `systemctl -i poweroff`, you can shutdown each computer remotely when SSH into the specific node. Same should apply for all other nodes.

### 7. Setup Hydra Process Manager

Hydra is a process management system used to manage and monitor processes on a Unix or Linux system. It is designed to be simple to use and can be used to start, stop, and monitor multiple processes at once. Hydra consists of two main components: a daemon process that runs in the background and a command-line interface that allows users to manage processes. The daemon process is responsible for starting and stopping processes, monitoring their status, and responding to commands from the command-line interface.

The Hydra Process Manager is included with the MPICH package, so we need to install MPICH on all computers. To do this, switch between each computer and use the following command to install it. To do this, make sure that the computer is connected to the internet.

```
sudo apt-get install mpich
```

This should install MPICH on all nodes. Once done, network all computers to the correct configuration and continue.

Next step is to configure MPICH. To do this, switch to the *master* and switch to *mpiuser*. (Use the command `su mpiuser`) Once in the *mpiuser*, move to the home directory. (Use the command `cd ~`) Now use a text editor to create a file called `hosts`, (Use the command `nano hosts` for *nano*) and enter the hostnames of the nodes you want the processes to run. You can either include the *master* node as a computer node or only use the other nodes. For example, refer below. Once done, save the file. (*Ctrl + X -> Y -> Enter* for nano)

```
master
node1
node2
```

The above example will run the programs distributed across *master*, *node1* and *node2*.

## 6. Running a Sample Program

### 1. Simple MPI Program in C

In order to write a simple MPI based program in C, we will first create the source code. For this, we will switch to the *mpiuser* (Use the command `su mpiuser`) and move into the home folder. (Use the command `cd ~`) Here, we will create a new file. (Use the command `nano hello.c`)

In it, we will write the following sample code.

```
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    // Initialize the MPI environment
    MPI_Init(NULL, NULL);

    // Get the number of processes
    int world_size;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);

    // Get the rank of the process
    int world_rank;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

    // Get the name of the processor
    char processor_name[MPI_MAX_PROCESSOR_NAME];
    int name_len;
    MPI_Get_processor_name(processor_name, &name_len);

    // Print off a hello world message
    printf("Hello world from processor %s, rank %d out of %d processors\n",
           processor_name, world_rank, world_size);

    // Finalize the MPI environment.
    MPI_Finalize();
}
```

The code is designed to print the node names and their ranks on the terminal of the *master*.

Next step is to compile the code into an object file. For this, use the following command.

```
mpicc -o hello.o hello.c
```

This will compile the code into an object file. Now we can run the MPI program on the cluster. For this, enter the following code.

```
mpiexec -n 12 -f hosts hello.c
```

Here, `-n 12` defines that the process should run on 12 threads and that `-f hosts` defines the file which contains the list of nodes to run the program on.

If working correctly, it should print a result as below.

```
Hello world from processor 1, rank 1 out of 12 processors
Hello world from processor 5, rank 5 out of 12 processors
Hello world from processor 3, rank 3 out of 12 processors
Hello world from processor 2, rank 2 out of 12 processors
Hello world from processor 8, rank 8 out of 12 processors
Hello world from processor 4, rank 4 out of 12 processors
Hello world from processor 6, rank 6 out of 12 processors
Hello world from processor 7, rank 7 out of 12 processors
Hello world from processor 9, rank 9 out of 12 processors
Hello world from processor 10, rank 10 out of 12 processors
Hello world from processor 12, rank 12 out of 12 processors
Hello world from processor 11, rank 11 out of 12 processors
```

### 2. Simple MPI Program in Python

It is also possible to run an MPI program on Python. For this, we will follow the same steps and create a source file, this time named `hello.py`. In it, we will write the following code.

```
from mpi4py import MPI

comm = MPI.COMM_WORLD
rank = comm.Get_rank()

print('My rank is ',rank)
```

Once done, save the file and use the following command to run the program.

```
mpirun -np 12 --hostfile hosts hello.py
```

## 7. Conclusion

In conclusion, a Beowulf cluster is a type of high-performance computing cluster that is built from commodity hardware and is designed to run parallel computing applications. The Beowulf cluster architecture is based on a master-slave model, where a central master node manages the workload and distributes tasks to multiple slave nodes.

MPI (Message Passing Interface) is a standard interface for programming parallel applications on a Beowulf cluster. It allows processes to communicate with each other and synchronize their actions across multiple nodes. MPI provides a set of standard libraries and functions that can be used to develop scalable and efficient parallel applications.

Together, a Beowulf cluster and MPI provide a powerful platform for running large-scale parallel applications. They can be used for a wide range of applications, including scientific simulations, data processing, and machine learning. The scalability and flexibility of these systems make them an ideal choice for organizations and research institutions that require high-performance computing capabilities.

## 8. Reference

- https://ubuntu.com/server/docs/network-configuration
- https://mpitutorial.com/tutorials/mpi-hello-world/
- https://stackoverflow.com/questions/32257375/how-to-run-a-basic-mpi4py-code
- https://www.open-mpi.org/doc/current/man1/mpirun.1.php
- https://www.programcreek.com/python/example/89108/mpi4py.MPI.Get_processor_name
- https://rabernat.github.io/research_computing/parallel-programming-with-mpi-for-python.html
- http://cs.boisestate.edu/~amit/research/beowulf/lab-notes/node19.html
- https://github.com/jbornschein/mpi4py-examples
- https://gist.github.com/arifsisman/bcb6409fcb6150099cb8ddd044e28060
- https://rabernat.github.io/research_computing/parallel-programming-with-mpi-for-python.html
