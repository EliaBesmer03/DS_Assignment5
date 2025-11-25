Assignment 5 Report
---------------------

# Team Members


# GitHub link to your (forked) repository

...

# Task 1

Note: Some questions require you to take screenshots. In that case, please join the screenshots and indicate in your answer which image refer to which screenshot.

1. What happens when Raft starts? Explain the process of electing a leader in the first
term.

Ans: 

2. Perform one request on the leader, wait until the leader is committed by all servers. Pause the simulation.
Then perform a new request on the leader. Take a screenshot, stop the leader and then resume the simulation.
Once, there is a new leader, perform a new request and then resume the previous leader. Once, this new request is committed by all servers, pause the simulation and take a screenshot. Explain what happened?

Ans: 

3. Using the same visualization, stop the current leader and two additional servers. After a few increments, pause the simulation and take a screenshot. Then resume all servers and restart the simulation. After the leader election, pause the simulation and take a screenshot. How do you explain the behavior of the system during the above exercise?

Ans: 

# Task 2

1. Which server is the leader? Can there be multiple leaders? Justify your answer using the statuses from the different servers.
Ans: 

2. Perform a PUT operation for the key "a" on the leader. Check the status of the different nodes. What changes have occurred and why (if any)?

Ans:

3. Perform an APPEND operation for the key "a" on the leader. Check the status of the different nodes. What changes have occurred and why (if any)?

Ans: 

4. Perform a GET operation for the key "a" on the leader. Check the status of the different nodes. What changes have occured and why (if any)?

Ans:



# Task 3

1. Shut down the server that acts as a leader. Report the status changes that you get from the servers that remain active after shutting down the leader. What is the new leader (if any)?

Ans:

1. Perform a PUT operation for the key "a" on the new leader. Then, restart the previous leader, and indicate the changes in status for the three servers. Indicate the result of a GET operation for the key "a" to the previous leader.

Ans:

3. Has the PUT operation been replicated? Indicate which steps lead to a new election and which ones do not. Justify your answer using the statuses returned by the servers.

Ans:

4. Shut down two servers: first shut down a server that is not the leader, then shut down the leader. Report the status changes of the remaining server and explain what happened.

Ans:

5. Can you perform GET, PUT, or APPEND operations in this system state? Justify your answer.

Ans:

6. Restart the servers and note down the changes in status. Describe what happened.

Ans:

## Network Partition

For the first experiment, create a network partition with 2 servers (including the leader)
on the first partition and the 3 other servers on the other one. Indicate the changes that occur in the status of a server on the first partition and a server on the second partition. Reconnect the partitions and indicate what happens. What are the similarities and differences between the implementation of Raft used by your key/value service (based on the PySyncObj library) and the one shown in the Secret Lives of Data illustration from Task 1? How do you justify the differences?

Ans:

For the second experiment, create a network partition with 3 servers (including the
leader) on the first partition and the 2 other servers on the other one. Indicate the changes that occur in the status of a server on the first partition and a server on the second partition. Reconnect the partitions and indicate what happens. How does the implementation of Raft used by your key/value service (based on the PySyncObj library) compare to the one shown in the Secret Lives of Data illustration from Task 1?

Ans: 


# Task 4

1. Raft uses a leader election algorithm based on randomized timeouts and majority voting, but other leader election algorithms exist. One of them is the bully algorithm, which is described in Section 5.4.1 of the Distributed Systems book by van Steen and Tanenbaum. Imagine you update the PySyncObject library to use the bully algorithm for Raft (as described in the Distributed Systems book) instead of randomized timeouts and majority voting. What would happen in the first network partition from Task 3?

Ans: 

2. Why is it that Raft cannot handle Byzantine failure? Explain in your own words.

Ans: 
