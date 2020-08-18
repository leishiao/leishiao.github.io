### Configure Quorum

1. In order for servers to contact each other, they need some contact information. 
2. To accomplish this, we are going to use the following configuration file:

```js
tickTime=2000
initLimit=10
syncLimit=5
dataDir=./data
clientPort=2181
server.1=172.41.15.75:2222:2223 
server.2=172.41.15.76:2222:2223
```

3. Each server.n entry specifies the address and port numbers used by ZooKeeper server n.
4. The second and third fields are TCP port numbers used for quorum communication and leader election.
5. We also need to set up some data directories. 
6. When we start up a server, it needs to know which server it is. 
7. A server figures out its ID by reading a file named `myid` in the data directory. 
8. We can create these files with the following commands: `echo id > data/myid`
9. When a server starts up, It obtains the server ID from `myid` and then uses the corresponding
   server.n entry to set up the ports it listens on.

**Problem**: How to specify id for each node in moss cluster. We need a bootstrap ?