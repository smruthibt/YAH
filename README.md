Yet Another Hadoop (YAH)

Yet Another Hadoop (YAH) is a Python-based, lightweight reimplementation of Hadoop’s HDFS and MapReduce.
It simulates a distributed file system (DFS) with Namenodes, Datanodes, replication, and job execution on a single machine using Flask servers and subprocess orchestration ￼.

Key Features

  •	Multiple DFS instances running simultaneously.
	•	Persistent storage (files remain across runs).
	•	Independent, real subprocesses for namenodes and datanodes.
	•	Fault tolerance:
	•	Handles deletion/failure of datanodes and namenodes.
	•	Secondary namenode failover.
	•	Namenode checkpointing.
	•	Robust client: retries and waits for backup namenode if primary is down.
	•	Basic MapReduce support on DFS files ￼.

System Components

1. DFS Initialization
	•	create_dfs.py
	•	Reads config file, validates all namenode/datanode paths.
	•	Generates dfs_setup_config.json.
	•	load_dfs.py
	•	Launches subprocesses for namenodes, datanodes, and client.
	•	Falls back to previous config if none is given.
	•	Assigns each node a unique port with a dedicated Flask server ￼.

2. Client
	•	Supports commands: put, cat, rm, mkdir, rmdir, ls
	•	Uses HTTP GET/POST requests to namenodes for metadata.
	•	Fetches metadata → issues requests to datanodes for file read/write/delete ￼.

3. Namenode
	•	Flask server for client RPCs.
	•	Primary namenode:
	•	Maintains edit log of operations.
	•	Periodically updated by the secondary into fsimage.
	•	Heartbeat thread monitors datanode availability.
	•	Failover: Secondary namenode promotes itself to primary if primary fails, then spawns a new secondary ￼.

4. Datanode
	•	Operations: read, write, delete.
	•	Read: returns file contents.
	•	Write/Delete: uses gossip protocol → update local state, update datanode list, propagate recursively ￼.

5. File System Image (fsimage)
	•	Represented as a directory tree on the underlying OS.
	•	Contains metadata only (not file data).
	•	Actual file blocks stored across datanodes ￼.

6. MapReduce
	•	Uses cat to fetch file contents from DFS.
	•	Executes map.py and reduce.py as subprocesses.
	•	Sorts intermediate data, pipes to reducer.
	•	Writes reducer output + job status back into DFS with put ￼.

Design Decisions
	•	Local filesystem as storage backend → simplifies path validity checks.
	•	Subprocesses for namenodes/datanodes → easier to simulate independent services.
	•	Flask REST APIs → lightweight inter-process communication.
	•	Checkpointing + failover → mimics Hadoop HA design.
	•	Gossip protocol → simple replication consistency mechanism.
	•	Temporary file for MapReduce output → integrates naturally with put semantics ￼.

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


===============================================================================================================================================================================================================================

Example Workflow
	1.	Run create_dfs.py → generates DFS config.
	2.	Run load_dfs.py → launches namenodes, datanodes, client.
	3.	Client issues put file.txt /user/data/:
	•	Sends metadata request to namenode.
	•	Namenode assigns datanodes.
	•	Client uploads via REST; datanodes replicate using gossip.
	4.	cat /user/data/file.txt → retrieves metadata + data blocks.
	5.	mapreduce_driver.py → runs map/sort/reduce and writes results into DFS ￼.
