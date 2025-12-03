# Cassandra_database_setup
I will setup 3 nodes cassandra cluster step by step. Anyone can setup by following this guidelines.


<!-- space -->

Background:

Apache Cassandra is a highly scalable, distributed NoSQL database designed to handle large amounts of data across many commodity servers with no single point of failure. It uses a peer-to-peer architecture where all nodes are equal, ensuring fault tolerance, high availability, and linear horizontal scalability.
Cassandra is optimized for write-heavy workloads, real-time analytics, time-series data, and applications that require continuous availability. It uses a masterless ring architecture, gossip protocol, and tunable consistency to provide flexible performance and reliability based on cluster design.


<!-- space -->

Key Features:

Masterless Architecture: Each node is equal; no master or coordinator bottleneck.

<!-- space -->
High Availability: Data is replicated across multiple nodes and datacenters.
<!-- space -->

Fault Tolerant: Automatically handles node failures without downtime.

<!-- space -->
Horizontal Scalability: Add or remove nodes without interrupting operations.

<!-- space -->
Tunable Consistency: Choose consistency levels per query (ONE, QUORUM, ALL).
<!-- space -->

Efficient for Write-Heavy Workloads: Designed for high-speed, low-latency writes.
<!-- space -->

Supports Multi-Datacenter Clusters: Rack and Data Centre-aware replication.

<!-- space -->

Common Use Cases:
<!-- space -->
Time-series data
<!-- space -->

IoT platforms
<!-- space -->

Logging and monitoring systems
<!-- space -->

Large-scale web applications
<!-- space -->

Analytics pipelines

<!-- space -->
Messaging platforms

<!-- space -->
Top 10 Companies Using Apache Cassandra:

<!-- space -->

Netflix,Apple,Uber,Meta,Spotify,Twitter,GitHub,eBay,American Express,Alibaba

<!-- space -->

Cluster Architecture (3 Nodes):
<!-- space -->
<img width="731" height="245" alt="image" src="https://github.com/user-attachments/assets/537c62dd-b48c-469d-9751-6f9a8d323b1c" />

<!-- space -->
Step 1: Prerequisites on all 3 nodes based on ubuntu 24 : Install openjdk  and cassandra
<!-- space -->
sudo apt install openjdk-17-jre-headless -y
<!-- space -->
sudo apt install cassandra -y
<!-- space -->
Step 2:  Check that cqlsh is working on all nodes
<!-- space -->
type cqlsh and press enter
<!-- space -->
step 3: Now edit the configuration and to do this, Stop the cassandra service on all 3 nodes.
<!-- space -->
sudo systemctl stop cassandra
<!-- space -->
step 4: Edit /etc/hosts on ALL nodes
<!-- space -->
sudo nano /etc/hosts
<!-- space -->
Add these lines but edit accordingly your own ip and name
<!-- space -->

192.168.178.63  cassandra1
<!-- space -->
192.168.178.65  cassandra2
<!-- space -->
192.168.178.67  cassandra3
<!-- space -->
save and exit

<!-- space -->

step 5: Update cassandra.yaml on each node:
<!-- space -->
sudo nano /etc/cassandra/cassandra.yaml
<!-- space -->
Change these values for each node:

<!-- space -->
Node1: 

cluster_name: 'mycluster'
<!-- space -->
num_tokens: 256
<!-- space -->
listen_address: 192.168.178.63
<!-- space -->
rpc_address: 0.0.0.0
<!-- space -->
rpc_broadcast_address: 192.168.178.63
<!-- space -->
<!-- space -->
seed_provider:
    <!-- space -->
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    <!-- space -->
      parameters:
     <!-- space -->
          - seeds: "192.168.178.63,192.168.178.65,192.168.178.67"
<!-- space -->
endpoint_snitch: GossipingPropertyFileSnitch
<!-- space -->


Node2:
<!-- space -->
cluster_name: 'mycluster'
<!-- space -->
num_tokens: 256
listen_address: 192.168.178.65
<!-- space -->
rpc_address: 0.0.0.0
<!-- space -->
rpc_broadcast_address: 192.168.178.65
<!-- space -->

seed_provider:
<!-- space -->
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    <!-- space -->
      parameters:
      <!-- space -->
          - seeds: "192.168.178.63,192.168.178.65,192.168.178.67"
          <!-- space -->

endpoint_snitch: GossipingPropertyFileSnitch

<!-- space -->

Node 3:
cluster_name: 'mycluster'
<!-- space -->
num_tokens: 256
<!-- space -->
listen_address: 192.168.178.67
<!-- space -->
rpc_address: 0.0.0.0
<!-- space -->
rpc_broadcast_address: 192.168.178.67
<!-- space -->

seed_provider:
<!-- space -->
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    <!-- space -->
      parameters:
      <!-- space -->
          - seeds: "192.168.178.63,192.168.178.65,192.168.178.67"
          <!-- space -->

endpoint_snitch: GossipingPropertyFileSnitch

<!-- space -->
step 6: Update rack/datacenter info (ALL nodes)
<!-- space -->
sudo nano /etc/cassandra/cassandra-rackdc.properties
<!-- space -->

For node 1 and 2:
<!-- space -->
dc=dc1-riad
<!-- space -->
rack=rack1

<!-- space -->
For node 3:
<!-- space -->
dc=dc1-riad
<!-- space -->
rack=rack2
<!-- space -->

Step 7: Allow ports in firewall (if enabled)

sudo ufw allow 7000/tcp
<!-- space -->
sudo ufw allow 7001/tcp
<!-- space -->
sudo ufw allow 7199/tcp
<!-- space -->
sudo ufw allow 9042/tcp
<!-- space -->
sudo ufw allow 9160/tcp
<!-- space -->

step 8: Start the cluster
<!-- space -->
start  node 1 first
<!-- space -->
sudo systemctl start cassandra
<!-- space -->
wait few seconds
<!-- space -->
Then start node2:sudo systemctl start cassandra
<!-- space -->
Then start node3:sudo systemctl start cassandra
<!-- space -->

step 9:  Verify cluster status
<!-- space -->
nodetool status
<!-- space -->
or if its showing authentication is required
<!-- space -->

nodetool -u cassandra -pw cassandra status

mrbhuiyan@mrbhuiyan-VirtualBox:~$ nodetool -u cassandra -pw cassandra status
Datacenter: dc1-riad
====================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address         Load        Tokens  Owns (effective)  Host ID                               Rack 
<!-- space -->
UN  192.168.178.63  113.82 KiB  256     100.0%            581790c7-615f-4b09-b337-36401f90c8df  rack1
<!-- space -->
UN  192.168.178.65  151.38 KiB  256     100.0%            e8377d23-e912-4e38-b6f1-927fa3ec3133  rack1
<!-- space -->
UN  192.168.178.67  79.71 KiB   256     100.0%            e226588f-75af-4069-bd73-359877e7af63  rack2

<!-- space -->

Step 10: Test CQL locally

type cqlsh and enter , you will get access to cql shell

cqlsh>

<!-- space -->

step 11: Create a keyspace with NetworkTopologyStrategy:

cqlsh> CREATE KEYSPACE testks
   ... WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};
<!-- space -->

  Step 12:   Create a table
cqlsh> CREATE TABLE test.items (id int PRIMARY KEY, value text);

<!-- space -->

Step 13: Insert data into table
cqlsh> INSERT INTO test.items (id, value) VALUES (1, 'hello');

<!-- space -->
qlsh> use testks;
cqlsh:testks> SELECT * FROM test.items;

 id | value
----+-------
  1 | hello

cqlsh:testks> INSERT INTO test.items (id, value) VALUES (2, 'hello1290345');
<!-- space -->
cqlsh:testks> INSERT INTO test.items (id, value) VALUES (3, 'hello12458345');
<!-- space -->
cqlsh:testks> INSERT INTO test.items (id, value) VALUES (4, 'hello123465');
<!-- space -->
cqlsh:testks> INSERT INTO test.items (id, value) VALUES (5, 'hello1234665');
<!-- space -->
cqlsh:testks> ELECT * FROM test.items

<!-- space -->

 id | value
 <!-- space -->
----+---------------
  5 |  hello1234665
  <!-- space -->
  1 |         hello
  <!-- space -->
  2 |  hello1290345
  <!-- space -->
  4 |   hello123465
  <!-- space -->
  3 | hello12458345


Here I create keyspace, table and insert data on node 1 but you can see those data is availabe on node2 as well as node 3. This is the magic of cassandra and this makes cassandra no down time!

<!-- space -->

mrhbhiyan@ubuntu2:~/Desktop$ cqlsh 
/usr/bin/cqlsh.py:477: DeprecationWarning: Legacy execution parameters will be removed in 4.0. Consider using execution profiles.
<!-- space -->
/usr/bin/cqlsh.py:507: DeprecationWarning: Setting the consistency level at the session level will be removed in 4.0. Consider using execution profiles and setting the desired consistency level to the EXEC_PROFILE_DEFAULT profile.
<!-- space -->
Connected to mycluster at 127.0.0.1:9042
[cqlsh 6.1.0 | Cassandra 4.1.10 | CQL spec 3.4.6 | Native protocol v5]
Use HELP for help.
<!-- space -->
cqlsh> SELECT * FROM test.items;

 id | value
----+-------
  1 | hello

(1 rows)

<!-- space -->


cqlsh> SELECT * FROM test.items;

 id | value
 <!-- space -->
----+---------------
  5 |  hello1234665
  <!-- space -->
  1 |         hello
  <!-- space -->
  2 |  hello1290345
  <!-- space -->
  4 |   hello123465
  <!-- space -->
  3 | hello12458345































