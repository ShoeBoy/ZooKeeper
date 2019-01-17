# ZooKeeper: FastLeaderElection

There are 3 classes in ZooKeeper that realize the interface "Election": "LeaderElection", "AuthFastLeaderElection", and "FastLeaderElection". 

The first 2 classes are abandoned after version 3.4.0. 

I'll introduce how FastLeaderElection works in ZooKeeper as follows. 

```text
currentVote = new Vote(myid, getLastLoggedZxid(), getCurrentEpoch());

this.electionAlg = createElectionAlgorithm(electionType);
```

Each node has a object "currentVote". 

Before starting, each node set the candidate as themselves by "startLeaderElection" in "QuorumPeer::start". 

A Thread "QuorumCnxManager.Listener" would be started when "FastLeaderElection" is being constructed. This thread is in charge of listening to the interface "ElectionAddr". 

The entrance of FastLeaderElection could be seen in "QuorumPeer::run". When the state of "QuorumPeer" is "LOOKING", "Election::lookForLeader" will be called to start the process. 

```text
private void sendNotifications() {
  for (QuorumServer server : self.getVotingView().values()) {
    long sid = server.id;
    ToSend notmsg = new ToSend(ToSend.mType.notification,
                    proposedLeader,
                    proposedZxid,
                    logicalclock,
                    QuorumPeer.ServerState.LOOKING,
                    sid,
                    proposedEpoch);
    sendqueue.offer(notmsg);
}
```

"FastLeaderElection::lookForLeader" establishes connection with other "PARTICIPANT" nodes by "sendNotifications". 

"sendNotifications" constructs a object "ToSend". "proposedLeader" stands for the sid of the candidate of the present node, while "proposedZxid" stands for the zxid of the candidate of the present node. In default, "logicalclock" works through "ZxidUtils.getEpochFromZxid\(newLeaderZxid\);", and is calculated according to zxid. I

When the object "ToSend" is added to the stack "sendqueue", there will be a independent thread "WorkSender" specifically in charge of sending "ToSend" to the node with corresponding sid, informing them the candidate conditions at present node. 

```text
//If wins the challenge, then close the new connection.
if (sid < self.getId()) {
  SendWorker sw = senderWorkerMap.get(sid);
  if (sw != null) {
    sw.finish();
  }
  closeSocket(sock);
  connectOne(sid);
} else {
  SendWorker sw = new SendWorker(sock, sid);
  RecvWorker rw = new RecvWorker(sock, sid, sw);

  sw.setRecv(rw);

  SendWorker vsw = senderWorkerMap.get(sid);
  if(vsw != null)
    vsw.finish();

  senderWorkerMap.put(sid, sw);

  if (!queueSendMap.containsKey(sid)) {
    queueSendMap.put(sid, new ArrayBlockingQueue<ByteBuffer>(
                        SEND_CAPACITY));
            }

  sw.start();
  rw.start();
}
```

When "QuorumCnxManager.Listener" receives the information, if the sid received is less than the sid of present node, the connection will be closed; otherwise, the socket connection will be maintained. 

In this way, although we've established node-to-node connections between every two nodes in the cluster, which leads to two sockets established between two nodes, but the socket from a node with the lower sid will be closed, which enables that the connection is is the only one exists. 

In a node-to-node connection, nodes will constantly receive "response" from other nodes. They would inform the source node their own candidate "currentVote" if the candidate in "response" is not a "PARTICIPANT" but an "OBSERVER". On the contrary, if the candidate in "response" is "PARTICIPANT", the node will make this "Message" into a package of an object "Notification", and put it into "recvqueue". 

"FastLeaderElection::lookForLeader" will constantly get "Notificaion" from "recvqueue", when the conditions "totalOrderPredicate" is satisfied: 

```text
protected boolean totalOrderPredicate(long newId, long newZxid, long newEpoch, long curId, long curZxid, long curEpoch) {
 return ((newEpoch > curEpoch) || 
                ((newEpoch == curEpoch) &&
                ((newZxid > curZxid) || ((newZxid == curZxid) && (newId > curId)))));
}
```

If the conditions "totalOrderPredicate" is satisfied, the candidate of other nodes are superior to the candidate of the present one. The superior candidate will be set as the present candidate by "update Proposal". 

```text
if (termPredicate(recvset, new Vote(proposedLeader, proposedZxid, logicalclock, proposedEpoch))) {
  Vote endVote = new Vote(proposedLeader, proposedZxid, logicalclock, proposedEpoch);
  leaveInstance(endVote);
  return endVote;
}
```

When we reach a majority vote as is described in Paxos algorithm, it is assumed that the election is over, and the FastLeaderElection process will be ended. 

