# Data Model in ZooKeeper
znode structures like a Unix file system. Each znode stores some information (key-value) like the current version of data changes in Znode, transaction Id of the latest transaction performed on the Znode. Since Zookeeper stores a tree like structure (znode) in the memory (in-memory database), each node has a limited size of the data (< 1MB).

Data access is atomic.

## Types of Znode
1. Ephemeral ZNode

    The lifetime of the ephemeral znode is dependant on the session. Once the session is terminated, the znode will be deleted automatically. Although the ephemeral znode is associated with a connection between client and zookeeper server, it is visible for all the clients.

    > Ephemeral znode may not have children, not even ephemeral ones.

2. Persistent Znode

    Persistent znode is only deleted by manual delete operation from client side.

3. Sequential Znode

    Client could ask to create a znode with a monotonically increase number as a suffix of the path. Each sibling sequential znode has a unique number from the parent's point of view.

## Watches
Watches allow clients to get notifications when a znode changes in some way. But watchers are triggered only once. To receive multiple notifications, a client needs to reregister the watch.

# ZooKeeper Services
The ZooKeeper service can run in two modes, **standalone mode** (development) and **replicated mode** (production). Conceptually, ZooKeeper's mission is to ensure that every modification to the tree of znodes is replicated to a majority of the ensemble. The other remaining replicas will eventually catch up with this state.

# Play with ZooKeeper
## Configuring ZooKeeper
Booting a zookeeper server needs a config file under `conf` subdirectory. Let's take a look at a sample config file in a three replicated ZooKeeper.
```bash
tickTime=2000 #time unit in ms used by Zookeeper
dataDir=./data/zookeeper1 #dir to store log info
clientPort=2181 #the port to listen for client connections
initLimit=5
syncLimit=2
#available server, the former port is for peer communication, the latter port is for leader election (TCP)
server.1=localhost:2881:3881
server.2=localhost:2882:3882
server.3=localhost:2888:3888
```
## Starting a ZooKeeper Server
Starting a standalon zookeeper server by running `bin/zkServer.sh start zoo.cfg`.

Check the status of a server: `bin/zkServer.sh status`. Alternatively, sening the four-letter words command to the client port using `nc`, like `echo srvr | nc 127.0.0.1 2181`.


## ZooKeeper command-line tools
ZooKeeper comes with a command-line tool for interacting with the        ZooKeeper namespace: `bin/zkCli.sh -server 127.0.0.1:2181`. There are nine basic operations in ZooKeeper.

|Operation|Description|
|--|--|
|create| create a znode(parent znode must already exist)|
| delete| delete a znode (must not have any childre)|
|exists|check if a znode exists and retreives its metadata|
|getACL, setACL| get/set the ACL|
|getChildren|get a list of the children of a znode|
|getData, setData| get/set the data associated with a znode|
|sync | sync a client's view of a znode|

# Use Case Stude
## Group Membership

- Dynamic Configuration
- Service Discovery
- Leader Election
- Distributed Lock
