# Demand Modeling

This page aims at describing the estimation of functions to be achieved by a centralized service for distributed systems.  

### Normal Demand 1: providing service. 

**use case name:** providing service

**scences:** server, client, implementing certain services

**use case description:** client makes a request; server finds a server in the server cluster to obtain the service 

**use case value:** client gets service, server provides service

**constraints and restrictions:** provde stable and reliable service to the reasonable requirements of the client; handle unreasonable requests from the client; switch to an appropriate server when current server breaks down

### Normal Demand 2: distributed lock. 

**use case name:** distributed lock 

**scences:** server, server controller, in a distributed system, the operations of reading, analyzing, and modifying data are dispersed to different nodes in the cluster, and the consistency of data operations needs to be ensured

**use case description:** distribute lock; read, analyze, and modify data; release lock

**use case value:** ensure data operation consistency

**constraints and restrictions:** correct acquisition and release of the lock

### Normal Demand 3: configuration management.

**use case name:** configuration management

**scences:** server, server controller, change configuration

**use case description:** initiate a configuration request; make a copy of the configuration file; configure all servers

**use case value:** ensure atomicity of synchronous operations; ensure that each server's configuration files are properly updated

**constraints and restrictions:** high-available configuration memory is needed for copying configuration files

### Exception 1: fault repair

**use case name:** fault repair

**scences:** server, server controller, troubleshoot distributed system

**use case description:** report failure; select a healthy node to be the master and learn about the health of each server in the cluster; after noticing a failing node, the master notifies the other servers in the cluster; master assigns computing tasks to different nodes; report the problem to admin to locate and fix the failure

**use case value:** handling faults, maintaining the system

**constraints and restrictions:** the main mechanism itself is robust and healthy

### Exception 2:  schizophrenia

**use case name:** schizophrenia

**scences:** server, server controller, two controllers appear

**use case description:** two controllers appear in the server cluster, giving commands to servers; number the leaders with an increasing sequence number to distinguish the new leader from the old ones; servers don't accept the instructions given by old leaders

**use case value:** avoid system confusion caused by leader replacement

**constraints and restrictions:** guaranteed data update for each server when leader replacement happens

### Alternative

**use case name:** alternative treatment

**scences:** server, server controller, the leader has failed

**use case description:** after the leader makes a decision, each server votes; if it gets agreements from more than half of the servers, execute the decision; if it gets agreements from less than half of the servers, re-elect the leader

**use case value:** ensure the availability of the leader

**constraints and restrictions:** avoid continuous execution failure when the system is normal; ensure that there is a server that could become a leader









