# ZooKeeper: client

### Environment:

ZooKeeper 3.4.9. 

### Entrance Function:

In bin/zkCli.sh, we could notice that the true entrance of client is a JAVA class "org.apache.zookeeper.ZooKeeperMain"

```text
"$JAVA" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" "-Dzookeeper.log.file=${ZOO_LOG_FILE}" \
     -cp "$CLASSPATH" $CLIENT_JVMFLAGS $JVMFLAGS \
     org.apache.zookeeper.ZooKeeperMain "$@"
```

```text
connectToZK(cl.getOption("server"));

while ((line = (String)readLine.invoke(console, getPrompt())) != null) {
  executeLine(line);
}
```

By reading the code, we could know that ZooKeeperMain are formed by 2 parts: 

1. construct an object "ZooKeeper" to establish connection with ZooKeeperServer
2. call class "jline.ConsoleReader" to read the input from terminal, and call interface "ZooKeeper"

As is described above, client is a simple package to "zookeeper.jar". After constructing an object "ZooKeeper", it calls the interface "ZooKeeper" to interact with server. 

### ZooKeeper Class

How the class ZooKeeper interacts with the server:

```text
public ZooKeeper(String connectString, int sessionTimeout, Watcher watcher,
            boolean canBeReadOnly)
        throws IOException 
{
  ConnectStringParser connectStringParser = new ConnectStringParser(connectString);
  HostProvider hostProvider = new StaticHostProvider( connectStringParser.getServerAddresses());
  cnxn = new ClientCnxn(connectStringParser.getChrootPath(),
  hostProvider, sessionTimeout, this, watchManager, getClientCnxnSocket(), canBeReadOnly);
  cnxn.start();
}
```

ZooKeeper uses the addrease of the server to create a class called "ClientCnxn". The system creates 2 threads in this class. 

```text
sendThread = new SendThread(clientCnxnSocket);
eventThread = new EventThread();
```

"SendThread" makes the requirements from ZooKeeper into a "Packet", send it to Server, and maintains the heartbeat with Server. 

"EventThread" analyses the "Response" received by "SendThread", and send it to "Watcher::processEvent" for further process. 

