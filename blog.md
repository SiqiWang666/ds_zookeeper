# Data Model in ZooKeeper
znode structures like a Unix file system. Each znode stores some information (key-value) like the current version of data changes in Znode, transaction Id of the latest transaction performed on the Znode. Since Zookeeper stores a tree like structure (znode) in the memory (in-memory database), each node has a limited size of the data (< 1M).

Types of znode:
1. Ephemeral ZNode

    The lifetime of the ephemeral znode is dependant on the session. Once the session is terminated, the znode will be deleted automatically. Although the ephemeral znode is associated with a connection between client and zookeeper server, it is visible for all the clients.

    > Ephemeral znode does not allow to have child node.

2. Persistent Znode

    Persistent znode is only deleted by manual delete operation from client side.

3. Sequential Znode

    Client could ask to create a znode with a sequential number as a suffix of the path. Each sibling sequential znode has a unique number from the parent's point of view.

# Use Case
- Dynamic Configuration
- Service Discovery
- Leader Election
- Distributed Lock

# Play with ZooKeeper
## Configuring ZooKeeper
Booting a zookeeper server needs a `.cfg` file. Let's take a look at a sample config file.
```bash
tickTime=2000 #time unit in ms used by Zookeeper
dataDir=./data/zookeeper1 #dir to store log info
clientPort=2181 #the port to listen for client connections
initLimit=5
syncLimit=2
#available server, the former port is for peer communication, the latter port is for leader election (TCP)
server.1=localhost:2881:3881
server.2=localhost:2882:3882
```
## Starting ZooKeeper Server
start a zookeeper server: `bin/zkServer.sh start zoo1.cfg`.

check the status of a server: `bin/zkServer.sh status zoo1.cfg`.

[todo]Stop a zookeeper server: `bin/zkServer.sh stop`.

## ZooKeeper CLI
We could use ZooKeeper CLI to connect with a server on the ZooKeeper cluster: `bin/zkCli.sh -server 127.0.0.1:2181`.


## Java API
It is better to use another thread to handle callback.

| ZoopKeeper API | sync | async|
|--|--|--|
|create| ✔︎| ✔︎|
|delete| ✔︎| ✔︎|
|exist| ✔︎| ✔︎|
|getData| ✔︎| ✔︎|
|setData| ✔︎| ✔︎|
|getACL| ✔︎| ✔︎|
|setACL| ✔︎| ✔︎|
|getChildren| ✔︎| ✔︎|
|sync| | ✔︎|
|createSession| ✔︎| |
|closeSession| ✔︎| |

# Ideas
Clients can set a watch on these Znodes and get notified if any changes occur in these znodes.