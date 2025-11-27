Assignment 5 Report
---------------------

# Team Members
Leon Schwendener
Elia Besmer

# GitHub link to your (forked) repository
https://github.com/EliaBesmer03/DS_Assignment5
...

# Task 1

Note: Some questions require you to take screenshots. In that case, please join the screenshots and indicate in your answer which image refer to which screenshot.

1. What happens when Raft starts? Explain the process of electing a leader in the first
term.

Ans: 
>When a Raft cluster starts, all servers begin in the follower state.
Since no leader exists yet and no heartbeats are received, each follower starts its election timeout timer.
Because the timeouts are randomized, one server’s timer expires first. This server becomes a candidate, increments the term, votes for itself, and sends RequestVote RPCs to all other servers.
If the candidate receives votes from a majority of the servers, it wins the election and becomes the leader for term 1.
The new leader immediately sends AppendEntries heartbeats to all followers to establish authority and to prevent further elections.
Thus, a single leader is elected in the first term, and the cluster becomes stable.

2. Perform one request on the leader, wait until the leader is committed by all servers. Pause the simulation.
Then perform a new request on the leader. Take a screenshot, stop the leader and then resume the simulation.
Once, there is a new leader, perform a new request and then resume the previous leader. Once, this new request is committed by all servers, pause the simulation and take a screenshot. Explain what happened?

Ans:
>After performing the first request, the leader appends the new log entry to its log and successfully replicates it to a majority of followers.
Once it is replicated on a majority, the leader marks the entry as committed, and all servers show this entry with a solid border (committed state).
When the simulation is paused and we perform a second request, this new entry appears only in the leader’s log, since the followers have not yet received it.
![task1_q2_1](task1_q2_1.png)
> 
>We then stop the leader, so it fails before it can replicate or commit this second entry.
Because followers no longer receive heartbeats, they start an election and eventually choose a new leader.
The new leader operates in a later term and accepts a new third request, appending this new entry to its own log and replicating it to a majority of servers.
This new entry becomes committed.
>When the old leader is restarted, it discovers that its term is outdated and steps down to follower state.
The new leader then brings the old leader’s log back into sync.
> - The uncommitted second entry (the one added just before the pause) is discarded, because it was never replicated or committed before the leader failed.
> - The committed entry from the new leader is replicated to the old leader so that all servers end with the same log.
> 
> In the final state, all servers contain the committed entries from the new leader, while the old leader’s uncommitted entry from the previous term is overwritten or removed according to Raft’s Log Matching and Leader Completeness rules.
![task1_q2_2](task1_q2_2.png) 

3. Using the same visualization, stop the current leader and two additional servers. After a few increments, pause the simulation and take a screenshot. Then resume all servers and restart the simulation. After the leader election, pause the simulation and take a screenshot. How do you explain the behavior of the system during the above exercise?

Ans: 
>When we stop the leader and two additional servers, only two servers remain running in the Raft cluster.
Because the original cluster size is five, a majority requires three servers.
With only two servers alive, no candidate can obtain a majority of votes, and therefore no leader can be elected.
> As a result, both remaining servers repeatedly increment their current term and start new elections, but all of these elections fail.
![task1_q3_1](task1_q3_1.png)
> Once all five servers are resumed and communication is restored, the system becomes able to form a majority again.
S1 still sends hearbeats to win the election with a majority.
After that, heartbeats stabilize the cluster, and the logs of previously stopped nodes begin synchronizing
![task1_q3_2](task1_q3_2.png)

# Task 2

1. Which server is the leader? Can there be multiple leaders? Justify your answer using the statuses from the different servers.
Ans: 
>By inspecting the /admin/status endpoint of all three servers, we observe:
> - node1 (8080) → "state": 0 (follower)
> - node2 (8081) → "state": 2 (leader)
> - node3 (8082) → "state": 0 (follower)
> 
> Therefore, node2 is the leader of the cluster. 
> 
> There cannot be multiple leaders in the same term because Raft guarantees Election Safety.
A server becomes leader only after receiving votes from a majority (2 out of 3), which prevents two servers from being leaders simultaneously.
All followers also report leader: TCPNode('node2:6001'), confirming the unique leader.

2. Perform a PUT operation for the key "a" on the leader. Check the status of the different nodes. What changes have occurred and why (if any)?

Ans:
>The log length increased from 2 → 3 on all nodes 
> Before the PUT, log_len was 2. 
> After the PUT:
> - node1: "log_len": 3
> - node2: "log_len": 3
> - node3: "log_len": 3 
> 
> This shows that the leader appended a new log entry and replicated it to both followers.
> 
> The commit index increased from 2 → 3
> 
> All nodes report:
> 
> "commit_idx": 3\
> "last_applied": 3
> 
> This means a majority of servers stored the new entry, so the leader marked it as committed, and all three nodes applied it to the state machine.

3. Perform an APPEND operation for the key "a" on the leader. Check the status of the different nodes. What changes have occurred and why (if any)?

Ans: 
>After performing APPEND("a", "mouse"), the commit index on all three servers increased from 3 to 4, and last_applied also became 4.
This shows that the leader appended a new log entry and successfully replicated it to a majority of the nodes.
As a result, all servers updated the value of key "a" from ["cat", "dog"] to ["cat", "dog", "mouse"].
Although the displayed log_len remained at 3, the operation was correctly committed and applied.
The request was sent to a follower (8080), which forwarded it to the leader (8081) as required by Raft.

4. Perform a GET operation for the key "a" on the leader. Check the status of the different nodes. What changes have occured and why (if any)?

Ans:
>After the GET operation, the log_len decreased from 3 to 2.
This is expected behavior in PySyncObj, because the library performs automatic log compaction.
Once all log entries are committed and applied (commit_idx = last_applied = 4), PySyncObj may remove or collapse older log entries to reduce memory usage.
GET itself does not create a log entry; it only triggered a compaction opportunity.
Therefore, the state remained correct while the visible log length became smaller.



# Task 3

1. Shut down the server that acts as a leader. Report the status changes that you get from the servers that remain active after shutting down the leader. What is the new leader (if any)?

Ans:
>After shutting down the original leader (node2), the remaining two servers (node1 and node3) continued to operate and still formed a majority of the cluster (2 out of 3).
Since no heartbeats were received from node2, both active servers started a new leader election.
Node1’s election timeout expired first, so it became the new leader in term 3, while node3 transitioned to the follower state.
>The /admin/status output confirms:
> - node1: state = 2 (leader), raft_term = 3, has_quorum = True 
> - node3: state = 0 (follower), leader = node1 
> - node2: unreachable (partner_node_status = 0)
> 
>Thus, a new leader was successfully elected because the cluster still had a majority after the failure of the original leader.

1. Perform a PUT operation for the key "a" on the new leader. Then, restart the previous leader, and indicate the changes in status for the three servers. Indicate the result of a GET operation for the key "a" to the previous leader.

Ans:
> eliabesmer@eduroamstud-10-255-185-188 kv % curl http://localhost:8081/keys/a
["cat","dog"]


3. Has the PUT operation been replicated? Indicate which steps lead to a new election and which ones do not. Justify your answer using the statuses returned by the servers.

Ans:
>Yes, the PUT operation was replicated.
All servers report commit_idx = 6 and last_applied = 6 after the restart of the old leader, showing that the new leader (node1) replicated the entry to both followers. 
> A new election occurred only when the original leader (node2) was shut down.
The PUT operation itself and the restart of the old leader did not trigger a new election, because node1 already had quorum and remained leader.

4. Shut down two servers: first shut down a server that is not the leader, then shut down the leader. Report the status changes of the remaining server and explain what happened.

Ans:
>After shutting down first a follower and then the leader, the remaining server (node3) stayed in follower state with has_quorum = False.
It still showed the old leader (node1) but both partner nodes were unreachable.
With only one server left, no majority can be formed, so no new leader can be elected and the system cannot make progress.

5. Can you perform GET, PUT, or APPEND operations in this system state? Justify your answer.

Ans:
>No in this state you cannot perform GET, PUT, or APPEND operations.
With only one server running, the system has no quorum, so it cannot elect a leader.
Without a leader:
> - PUT and APPEND cannot be accepted (writes require a leader and a majority). 
> - GET also fails because PySyncObj does not serve read requests when the node is in follower state without a leader.
> 
>Therefore, the system is unavailable until at least two servers are running again and a leader can be elected.

6. Restart the servers and note down the changes in status. Describe what happened.

Ans:
>After restarting the two failed servers (node2 first, then node1), node3, the only server that had remained running, immediately became the leader in term 4.
Once node1 and node2 rejoined, they detected the higher term and reverted to follower state.
Node3 then brought both nodes fully up to date by sending the missing log entries, so all servers now show:
> - commit_idx = 7 
> - last_applied = 7 
> - identical key/value state 
>
> The cluster reached a stable state again with node3 as leader and full quorum restored.

## Network Partition

For the first experiment, create a network partition with 2 servers (including the leader)
on the first partition and the 3 other servers on the other one. Indicate the changes that occur in the status of a server on the first partition and a server on the second partition. Reconnect the partitions and indicate what happens. What are the similarities and differences between the implementation of Raft used by your key/value service (based on the PySyncObj library) and the one shown in the Secret Lives of Data illustration from Task 1? How do you justify the differences?

Ans:
>Similarities 
> - Only the majority partition (3 servers) can elect a leader — exactly like in the Secret Lives of Data Raft demo. 
> - The 2-server minority partition cannot elect a leader because it has no quorum. 
> - After reconnecting the network, the minority nodes step down and synchronize their logs with the majority leader. 
> - The system returns to one single leader and a consistent state.
> 
>Differences 
> - PySyncObj increases the term numbers much faster because it uses shorter and more aggressive election timeouts. 
> - PySyncObj performs automatic log compaction, so log_len may shrink or look different from the demo. 
> - The Secret Lives of Data illustration shows Raft very slowly and clearly, while PySyncObj batches and optimizes internal messages, making transitions look less clean. 
>
>Justification 
> - Secret Lives of Data is a teaching visualization that shows Raft step-by-step. 
> - PySyncObj is a performance-optimized implementation, so it compresses logs, retries elections quickly, and hides low-level details. 
> - The differences are due to practical optimizations, but the core Raft behavior (majority wins, minority stalls, logs synchronize after reconnect) remains the same.

For the second experiment, create a network partition with 3 servers (including the
leader) on the first partition and the 2 other servers on the other one. Indicate the changes that occur in the status of a server on the first partition and a server on the second partition. Reconnect the partitions and indicate what happens. How does the implementation of Raft used by your key/value service (based on the PySyncObj library) compare to the one shown in the Secret Lives of Data illustration from Task 1?

Ans: 
>Similarities 
> - Only the majority partition keeps or elects a leader. 
> - The minority partition cannot make progress. 
> - After reconnecting, minority nodes accept the leader and synchronize.
>
>Differences 
> - PySyncObj shows faster elections and rapid term increases because of short timeouts. 
> - It performs automatic log compaction, so log_len differs from the demo. 
> - Transitions look less „smooth“, because PySyncObj batches RPCs and prioritizes speed over visualization clarity.
> 
> Secret Lives of Data shows Raft in a simple, slow, step-by-step manner.
PySyncObj is an optimized production library, so timing, term increments, and log presentation behave differently.
The core Raft rules, however, remain correct.

# Task 4

1. Raft uses a leader election algorithm based on randomized timeouts and majority voting, but other leader election algorithms exist. One of them is the bully algorithm, which is described in Section 5.4.1 of the Distributed Systems book by van Steen and Tanenbaum. Imagine you update the PySyncObject library to use the bully algorithm for Raft (as described in the Distributed Systems book) instead of randomized timeouts and majority voting. What would happen in the first network partition from Task 3?

Ans:
>If Raft used the bully algorithm instead of randomized timeouts and majority voting, the first network partition would result in multiple leaders.
In the bully algorithm, the node with the highest ID declares itself leader as soon as it cannot reach higher-ID nodes. So it does not require a majority.
In the first partition (2 servers including the former leader on one side, and 3 servers on the other side), both partitions would contain a node with the highest ID in their respective subset.
Each of these nodes would independently elect itself leader. 
> As a result:
> - Partition A elects a leader 
> - Partition B elects a different leader 
> - Split-brain situation 
> - Conflicting logs, lost writes, inconsistency 
> 
> This violates Raft’s core safety property, which strictly requires a majority to elect a leader.

2. Why is it that Raft cannot handle Byzantine failure? Explain in your own words.

Ans: 
>Raft cannot handle Byzantine failures because it assumes that nodes may fail by crashing or becoming unreachable, but never by behaving maliciously or sending incorrect or conflicting messages.
Raft’s correctness depends on servers exchanging truthful votes and log entries.
