# Leader Election

| Role | Description |
| :--- | :--- |
| Leader | Leader is responsible for the initiation and resolution of the vote, and updating system status.  |
| Learner: Follower | Followers are used to receive requests from clients and send back results to the clients.  |
| Learner: Observer | Observers could connect with the client, and forward the request to the leader. However, the observer does not participate in the voting process and only synchronizes the status of the leader. The purpose of the observer is to expand the system and increase the reading speed.  |
| Client | Client is where requests originate from.  |

