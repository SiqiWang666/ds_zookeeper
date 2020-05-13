# ZooKeeper Overview
Apache ZooKeeper is a library that provides coordination service in distributed system.

# Data Model in ZooKeeper
Znode structures like a Unix file system. Each znode stores some information (key-value) like the current version of data changes in Znode, the latest transaction ID. Since Zookeeper stores a tree like structure (znode) in the memory, each node has a limited size of the data (< 1MB).

## Types of Znode
1. Ephemeral ZNode

    The lifetime of the ephemeral znode is dependant on the session. Once the session is terminated, the znode will be deleted automatically. Although the ephemeral znode is associated with a connection between client and zookeeper server, it is visible for all the clients.

    > Ephemeral znode may not have children, not even ephemeral ones.

2. Persistent Znode

    Persistent znode is only deleted by manual delete operation from client side.

3. Sequential Znode

    Client could ask to create a znode with a monotonically increase number as a suffix of the path. Each sibling sequential znode has a unique number from the parent's point of view.

## Watches
A ZooKeeper Watcher object allow clients to get notifications when a znode changes in some way and get notifications of changes in the ZooKeeper state. But watchers are triggered only once. To receive multiple notifications, a client needs to reregister the watch.

# ZooKeeper Services
The ZooKeeper service can run in two modes, **standalone mode** (development) and **replicated mode** (production). Conceptually, ZooKeeper's mission is to ensure that every modification to the tree of znodes is replicated to a majority of the ensemble. The other remaining replicas will eventually catch up with this state.

## Leader Election
ZooKeeper servers in an ensemble go through a process of electing a distinguished member, called the *leader*. The other machines are termed *followers*. Leader election is very fast, around 200 ms.

## Atomic Broadcast
All write requests all forwarded to the leader, which broadcasts the update to the followers. When a majority have persisted the change, the leader commits the update, and the client gets a response saying the update succeeded.

Every update made to the znode tree is given a globally unique identifier, called a **zxid**, which is ordered. If a `zxid z1` is less than `zxid z2`, then `z1` happened before `z2`.

## Sessions
❓A ZooKeeper client is configured with the list of servers in the ensemble. Once a connection has been established with a ZooKeeper server, the server creates a new session for the client. Sessions are kept alive by the client sending ping requests(aka heart-beats) whenever the session is idle for longer than a certain period. This is done automatically by the client library.

**Tick time** is the very basic period of time in ZooKeeper. Other settings are defined in terms of tick time.  The session timeout, for example, may not be less than 2 ticks or more than 20.

# Play with ZooKeeper
## Configuring ZooKeeper
Each server in the ensemble has a numeric identifier that is unique within the ensemble and must fall between 1 and 255. The number is specified in plain text in a file named `myid` in the dir specified by `dataDir` property. A ZooKeeper configuration file include all the servers in the ensemble. Each server is configued as `server.n=hostname:port:port`. The first port is used for followers to connect to the leader. The second one is used for leader election.

Let's take a look at a sample config file for a three replicated ZooKeeper.
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
When a ZooKeeper server is booted up, it reads its id from `myid` file and then reads the config file to determin the ports it should listen on and discover the address of other servers in the ensemble.

Starting a standalon zookeeper server by running `bin/zkServer.sh start zoo.cfg`.

Check the status of a server: `bin/zkServer.sh status`. Alternatively, sending the four-letter words command to the client port using `nc`, like `echo srvr | nc 127.0.0.1 2181`.


## ZooKeeper command-line tools
ZooKeeper comes with a command-line tool for interacting with the ZooKeeper namespace: `bin/zkCli.sh -server`. There are nine basic operations in ZooKeeper.

|Operation|Description|
|--|--|
|create| create a znode(parent znode must already exist)|
| delete| delete a znode (must not have any childre)|
|exists|check if a znode exists and retreives its metadata|
|getACL, setACL| get/set the ACL|
|getChildren|get a list of the children of a znode|
|getData, setData| get/set the data associated with a znode|
|sync | sync a client's view of a znode|

# Use Case Study

## Dynamic Configuration
ZooKeeper can act as a hightly available store for configuration, allowing application participants to retrieve or update configuration files.

## Service Discovery/Service Membership
## Distributed Lock
