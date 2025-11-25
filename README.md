# Fault-Tolerant Key/Value Service


## Project Structure

```bash
docker #A folder with the scripts and config files for Docker
└── partition #A folder for the scripts and Docker config files related to the network partition
images #A folder where to put the screenshots that you generate.
kv #A folder where to implement the key/value service
└── kvstorage.py #A Python script to implement an instance of the Key/Value Service. TODO: complete
└── kvstorage_http.py #A Python script to implement an HTTP server to interact with an instance of the Key/Value Service. TODO: complete
api_description.yml #The API description of the server to implement
.python-version #The version of Python used. This file is used by uv
pyproject.toml # A file used  to provide the configuration for uv.
uv.lock # A file to indicate the packages used by uv.
README.md #This README.
Report.md #The Report to complete
```

## Setting

The Python [`Library PySyncObj`](https://readthedocs.org/projects/pysyncobj/) will be used through the Assignment.


## Task 1: Discovering Raft

Follow the explanation of Raft presented at: https://thesecretlivesofdata.com/raft/.

The website https://raft.github.io/ shows an interactive visualization of how Raft works for a group of 5 servers, supporting a simple database with only one type of transaction.

Answer the questions about it in your Report.

## Task 2: Implementation of a Fault-Tolerant Key/Value Service with Raft

In this task, you should complete the Python scripts [`kvstorage.py`](kv/kvstorage.py)  and [`kvtosrage_http.py`](kv/kvstorage_http.py), in the kv folder.

[`kvstorage.py`](kv/kvstorage.py)  is an implementation of a Raft server, implementing a key/value store. 

[`kvstorage_http.py`](kv/kvstorage_http.py) is an HTTP server that enables interactions with a Raft server from kvstorage.py.

### Running the servers

Using Docker, you can also run all 3 servers at once using the command, in the folder [`docker`](docker):

```bash
docker compose up --build -d
```

Then, to stop the containers and the network:

```bash
docker compose down
```

The API description for the HTTP servers is provided in [`api_description.yml`](api_description.yml)

You can perform an example PUT operation to the key/value service for key "a" with: 

```bash
curl -X POST http://localhost:8080/keys/a \
  -H "Content-Type: application/json" \
  -d '{"type": "PUT", "value": ["cat", "dog"]}'
```

You can perform an APPEND operation to the key/value service for key "a" with: 

```bash
curl -X POST http://localhost:8080/keys/a \
  -H "Content-Type: application/json" \
  -d '{"type": "APPEND", "value": "mouse"}'
```

You can perform a GET operation to the key/value service for key "a" with:

```bash
curl http://localhost:8080/keys/a
```

You can also perform this operation to another node to see that it has been replicated with (for example):

```bash
curl http://localhost:8081/keys/a
```

You can retrieve the status of node 0 with:

```bash
curl http://localhost:8080/admin/status
```

Note: GET requests can also be seen directly in the browser.

Complete Task 2 in the Report

IMPORTANT: Docker uses its own DNS system, that resolves container names to IP addresses. You can see this in the /admin/status for a server. 


# Task 3: Evaluation of the Key/Value Service



## Shutting down servers

Run the command in [`docker`](docker) for appropriate values of i:
```bash
docker compose stop node{i}
```

To restart the node:
```bash
docker compose restart node{i}
```


## Network partition

In the [``docker/partition``](docker/partition) folder, run 

```bash
docker compose up --build -d
```

to start the 5 servers.

To disconnect a server from the network, use, in the [``docker/partition``](docker/partition) folder (update the parameters for the actual partition): 

```bash
./partition.sh "node1 node2 node3" "node4 node5"
```

Note: The first strings should indicate the nodes for the first partition and the second string indicates the nodes for the second partition. The partition indicates to containers to reject packets from containers that are not located in their partition, effectively implementing a network partition. 

To reconnect the server afterwards, run, in the [``docker/partition``](docker/partition) folder:

```bash
./reconnect.sh
```

Note: On Windows, both scripts need to be run using the Windows Subsystem for Linux v2 (WSL2). On Linux and WSL2, you need to run these scripts as root users. On WSL2, you may need to first convert the two scripts with the proper line endings, using: ```dos2unix partition.sh``` and ```dos2unix reconnect.sh```

When, you no longer need to use the containers, stop the containers and the network by running, in [``docker/partition``](docker/partition):
```bash
docker compose down
```

Complete Task 3 in the Report. You should compare what you observe with the network partition presented at: https://thesecretlivesofdata.com/raft/.


# Task 4: Questions on Raft

Complete Task 4 in the Report.
