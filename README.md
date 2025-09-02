Yet Another Hadoop (YAH)

Yet Another Hadoop (YAH) is a Python-based, lightweight reimplementation of Hadoop’s HDFS and MapReduce.
It simulates a distributed file system (DFS) with Namenodes, Datanodes, replication, and job execution on a single machine using Flask servers and subprocess orchestration.

<img width="675" height="533" alt="image" src="https://github.com/user-attachments/assets/a59d4aba-87e8-4584-922e-cb3106b196ae" />

Key Features
1. Multiple DFS instances running simultaneously.
2. Persistent storage (files remain across runs).
3. Independent, real subprocesses for namenodes and datanodes.
4. Fault tolerance:
5. Handles deletion/failure of datanodes and namenodes.
6. Secondary namenode failover.
7. Namenode checkpointing.
8. Robust client: retries and waits for backup namenode if primary is down.
9. Basic MapReduce support on DFS files.

System Components

1. DFS Initialization
	1. create_dfs.py
 	2. Reads config file, validates all namenode/datanode paths.
	3. Generates dfs_setup_config.json.
	4. load_dfs.py
	5. Launches subprocesses for namenodes, datanodes, and client.
	6. Falls back to previous config if none is given.
	7. Assigns each node a unique port with a dedicated Flask server ￼.

2. Client
    1. Supports commands: put, cat, rm, mkdir, rmdir, ls
	2. Uses HTTP GET/POST requests to namenodes for metadata.
	3. Fetches metadata → issues requests to datanodes for file read/write/delete ￼.

3. Namenode
    1. Flask server for client RPCs.
	2. Primary namenode:
	3. Maintains edit log of operations.
	4. Periodically updated by the secondary into fsimage.
	5. Heartbeat thread monitors datanode availability.
	6. Failover: Secondary namenode promotes itself to primary if primary fails, then spawns a new secondary ￼.

4. Datanode
    1. Operations: read, write, delete.
	2. Read: returns file contents.
	3. Write/Delete: uses gossip protocol → update local state, update datanode list, propagate recursively ￼.

5. File System Image (fsimage)
    1. Represented as a directory tree on the underlying OS.
	2. Contains metadata only (not file data).
	3. Actual file blocks stored across datanodes ￼.

6. MapReduce
    1. Uses cat to fetch file contents from DFS.
	2. Executes map.py and reduce.py as subprocesses.
	3. Sorts intermediate data, pipes to reducer.
	4. Writes reducer output + job status back into DFS with put ￼.

Design Decisions
1. Local filesystem as storage backend → simplifies path validity checks.
2. Subprocesses for namenodes/datanodes → easier to simulate independent services.
3. Flask REST APIs → lightweight inter-process communication.
4. Checkpointing + failover → mimics Hadoop HA design.
5. Gossip protocol → simple replication consistency mechanism.
6. Temporary file for MapReduce output → integrates naturally with put semantics.

Quick Start

Install dependencies
pip install -r requirements.txt
Initialize DFS
python create_dfs.py
python load_dfs.py --config dfs_setup_config.json

Run client commands

put local.txt /hdfs/data/
ls /hdfs/data/
cat /hdfs/data/local.txt
rm /hdfs/data/local.txt

Run MapReduce

python mapreduce_driver.py input.txt output/


Example Workflow
1.	Run create_dfs.py → generates DFS config.
2.	Run load_dfs.py → launches namenodes, datanodes, client.
3.	Client issues put file.txt /user/data/:
	1. Sends metadata request to namenode.
	2. Namenode assigns datanodes.
	3. Client uploads via REST; datanodes replicate using gossip.
4.	cat /user/data/file.txt → retrieves metadata + data blocks.
5.	mapreduce_driver.py → runs map/sort/reduce and writes results into DFS.
