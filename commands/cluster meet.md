`CLUSTER MEET` is used in order to connect different Redis nodes with cluster
support enabled, into a working cluster.

The basic idea is that nodes by default don't trust each other, and are
considered unknown, so that it is unlikely that different cluster nodes will
mix into a single one because of system administration errors or network
addresses modifications.

So in order for a given node to accept another one into the list of nodes
composing a Redis Cluster, there are only two ways:

1. The system administrator sends a `CLUSTER MEET` command to force a node to meet another one.
2. An already known node sends a list of nodes in the gossip section that we are not aware of. If the receiving node trusts the sending node as a known node, it will process the gossip section and send an handshake to the nodes that are still not known.

Note that Redis Cluster forms a full mesh, but it is not needed to send as much as `CLUSTER MEET` commands as needed to form the full mesh, because thanks to gossiping the missing links will be created.

For example if we imagine a cluster formed of the following four nodes called A, B, C and D, we may send just the following set of commands to A:

1. CLUSTER MEET B-ip B-port
2. CLUSTER MEET C-ip C-port
3. CLUSTER MEET D-ip D-port

As a side effect of `A` knowing and being known by all the other nodes, it will send gossip sections in the heartbeat packets that will allow each other node to create a link with each other one, forming a full mesh in a matter of seconds, even if the cluster is large.

## Implementation details: MEET and PING packets

When a given node receives a `CLUSTER MEET` message, the node specified in the
command still does not know the node we sent the command to. So in order for
the node to force the receiver to accept it as a trusted node, it sends a
`MEET` packet instead of a `PING` packet. The two packets have exactly the
same format, but the former forces the receiver to acknowledge the node as
trusted.

@return

@simple-string-reply: `OK` if the command was successful. If the address or port specified are invalid an error is returned.