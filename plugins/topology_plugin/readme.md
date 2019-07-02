#Topology Plugin
This optional plugin facilitates the gathering of network performance metrics for nodes and links. A second plugin, topology api plugin, adds some http commands used to fetch network status info or to add or remove watchers interested in tracking some of the data.

The topology plugin is tied closely to the net plugin, adding a new timer along with an optional reference to the topology plugin and local node and link ids that are globally unique. A new topology message is added to the net protocol list which is used to communicate changes to the network map, periodic updates of perfomance data, or changes of interested watchers.

##watching the network
this is the way to receive periodic updates on performance metrics. Although each node will have an entire map of the network, each node only has the performance metrics for itself and its direct peers. Watcher subscriptions are shared over the network so that the buden of updating watchers is spread acreoss the network.

A watcher is defined by a UDP endpoint along with a list of link ids it is interested in and a specification of which data points it is interested in. watchers are updated by the node that owns the passive connection side of the identified link. 

##identifying nodes and links
The topology plugin maintains a map of the p2p network in terms of nodes, which are instances of nodeos, and the p2p connections between them. Since in many cases nodes, particularly block producing nodes, are sequestered behind firewalls and using otherwise nontroutable endpoints nodes are identified by a hash of a strinified collection of p2p host and port, and an arbitrary "bp identifier" string which is configured via the command line or config.ini file. Additional node identifiers include the role of the particular node and any specific producer account names, if any. 

Links are defined by the pair of nodes being connected, which node is the active or passive connectors. Link descriptors also include the number of hops between the nodes and the kind of data the link is used for. The number of hops in this case refers to things such as network bridges, routers, firewalls, load balancers, proxies, or any other layer 3 or lower devices. Intermediate eosio nodes do not count since a link is terminated at a nodeos instance. 

The topology plugin will produce a graphvis compatible rendition of the network map to help operators visualize the possible areas of network congestion that leads to missed blocks and microforks.

Ultimately this map could help further reduce network traffic by not forwarding unblocked transactions to _all_ nodes, preferring to route them only to producer nodes which might be responsible for enblocking or verifying. Likewise new block messages can be routed in a way that avoids duplication.
##API
this table highlights the calls available using the http interface

| request |   arguments   |   result   | description | 
|---------|---------------|------------|-------------|
 nodes | string: in\_roles | vector of node descriptors | Nodes have specific roles in the network, producers, api, backup, proxy, etc. This function allows you to retrieve a list of nodes on the tenwork for a specific role, or for any role. 
  links | string: with_node | vector of link descriptors | supply a [list of?] node ideentifies as retrived with the /nodes/ function and get back a list of descriptors covering all of the connections to or from the identified node
   watch | string: udp\_addr string: links string: metrics | void | supply a watcher callback address for metrics related to the specified links
  unwatch | string: udp\_addr string: links string: metrics | void | remove a watcher callback address for metrics related to the specified links
 
##source file map
The plugin follows the common idiom of a lightweight interface class and a more robust hidden implementation class. To improve the atomicity of included headers, these are broken apart as shown

| filename | definitions | description    |
|----------|-------------|----------------|
 topology_plugin.hpp | topology\_plugin | The accessable interface for the module 
 messages.hpp | watcher | definition of a watcher 
 | link\_sample | a collection of metrics related to a single link id.
 | map\_update | a collection of updates to the relationships of nodes and links, for example when a new nodeos instance becomes active or one goes down.
 | topo\_data | an amalgam of the previous 3 data types 
 | topology\_message | the new p2p message type used to communicate network status information between the various peers. 
 link\_descriptor.hpp | link\_roles | an enumeration of possible specialized use for this link. This could be blocks, transactions, commands, or any combination of those three. 
 | link\_descriptor | the verbose description of the link as defined by the connected nodes. This gets stringified and hashed to generate a universally unique, but otherwise anonymous identifier.
 node\_descriptor | node\_descriptor | the verbose description of a node used to generate a globally unique node id.
 
 The implementation details are entirely contained in a single source file for now, topology\_plugin.cpp. The types defined in this source file are described here.
 
|  typename  |  description  |
|------------|---------------|
   topo\_node | contains the verbose description of the node along with it's hashed id along with a list of no