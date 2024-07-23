
The underlying concept in Federated Distribution is to retain as much as possible of that simplicity. If each cluster of nodes ran an independent instance of the application it would be that simple - each having its own set of users with their own data to worry about but that would run contrary to Zyx's objectives. Inspired by its namesakes in administrative and geopolitical system Federated Distribution allows each region to operate independently but provides a structured mechanism to coordinate and exchange data between regions, effectively combining the independent regions under one umbrella. 

It's both impossible and unnecessary to know in advance what proportion of any region's data would only ever be referenced within the region versus the what would be referenced across region boundaries. Impossible because it depends purely on what data users add and how well it is received, but unnecessary because data that is referenced across region boundaries evidently have (more) value to users in other regions so the cost of replicating those records to the regions where they are needed is warranted. What is possible, necessary and important is to make provision for such data to be identified automatically, dynamically replicated as required without overrunning available resources and of course monitored so that additional resources can be deployed to where it would be most effective.

Also aligned with real-world federation concepts is Federal Distribution's acknowledgement that regions are neither static nor sized equally. Every region grows at its own rate, meaning it's almost certain that no two will be the same size at the same time. As they grow beyond the region's provisioned capacity the users in that region need additional resources either in the form of more nodes to the cluster or additional clusters. The latter would require that there are data centres closer to some users in the original region that could host a cluster that's closer to those users. But if access times would be unaffected it would make more sense to grow the number of nodes in the cluster sharing the workload.

### Database IDs

As direct consequence (or enabler, depending on your perspective) of the Federated Distribution approach it emerges as essential that every fragment of "federated" data has something about it that binds it to its home or owner region. Though users are also bound to one region at a time which can change over time there is no room for ambiguity with respect to which region owns which fragment of data. In bona fide distributed databases such as [YugabyteDB](https://www.yugabyte.com) this can lead to incredibly complex configurations especially when it starts involving [remote clusters](https://www.yugabyte.com/blog/tag/xcluster/). FeDist has simplicity as a primary objective and that is best served when the home region of each fragment of data forms part of its key or identifier. For Zyx that means that the 64 bit (bigint) identifier it uses for its main data tables are split into 16 bits for a region id and 48 bits as generated (sequence) identifier unique within the region. Configured like that, any data reference (foreign key) encountered in code can be inspected quickly separated into those available locally and those that needs to come from which other region(s). Which means that the applications's business logic at that point need not distinguish and can deal with all data the same and let the data access layer resolve the local and remote distinction.

This identifier, in particular the 16 bits of region identifier it contains is the reference implementation for Zyx's choice of delineator. The globe being recursively subdivided into smaller and smaller subregions in accordance with where active users are is not something Zyx cam change so adopting a dynamic hierarchy of regions as delineator into the application as its fundamental delineator for data- and process- distribution allows Zyx to match its distributed facilities very accurately to the reality of its environment.

### Region Hierarchy

In case it wasn’t' clear above it bears getting highlighted that 16 bits worth of region number is a lot of regions. We also saw that for the application to respond accurately to where and when user numbers grow we don't want a flat list of non-overlapping regions but rather one global region that keeps subdividing into smaller regions as required resulting in a hierarchical region structure. This structure is going to become critically important when we tackle inter-region communication because if there's one lesson we learnt well from the built-in distribution facilities it is that a full mesh between all nodes doesn't scale well when thousands of nodes are involved.  

### Separation of Concerns

Given the importance of region identifiers and region hierarchy in the application’s design it would not appear out of place for the application to implement what it needs for distributed processing and data storage from first principles using low level libraries.

At the other end of the scale the application's designers might choose a powerful high level library which comes with its own approach to process- and data-distribution that the application design has to confirm with if. 

The more beneficial middle ground is to isolate delineation as a separate concern which the application supplies in the form of a defined interface which the library then calls to accurately inform the decisions it makes regarding data- and process-distribution.
 
The particular way we arrange for the FeDist library to adopt how the application has mapped between its reality and its data model is key to being able to use FeDist for any purpose beyond the reference example.  It warrants taking a bit more space and energy to convey how that mechanism works. This is where the example set by the reference implementation is helpful as it allows us to have the discussion about real world situations rather than some abstract scenarios.


### What a Mesh!

Within a region's cluster the default full mesh network is probably still good for our range of scaling requirements but to fully mesh all the nodes in a multi-cluster setup or even to fully mesh betwen clusters is not practical. To our target audience, i.e. applications like Zyx that solve an inherently federated problem would prefer to see all regions as connected without getting involved in the details. The real messy part is that the data about which users are served from which regions and which data belongs to which user and region are all application concerns but that data also forms the basis of what the FeDist library needs to set up efficient connections between regions.

The application, or more specifically the humans who provision and configure the regional clusters where it runs in response to demans, is the source of region data, provided in the form of clsuter configuration values.

Each cluster is configured with at least two sets of detail - its own region id, parent region and public ip address or DNS name, and similar details for either the root of the region hierarchy or that of an intermediate node which serves as some kind of regional hub. When the regional cluster initialises itself it sets up its local database to generate identifiers in the range given by its allocated region id, primes the tests for local data with its regional id and makes contact with the designated parent/root region to announce itself as functional. The root (or parent if so chosen by the developers) then ensures that all other regions get updated with the new region's contact details and once that is done the new region would be allowed to start requesting data from any other region as well as store locally generated data in its own database. 

Simply put, every region has a local copy of the region registry which is being kept consistent throughout the federated system by a simple master-slave replication mechanism. The data is tiny compared to the application's primary payload and does not change often.

But the data required to decide on an appropriate balance between resource consumption and minimising delays due to latency and server load is quite a different matter. For this additional data we're talking about a set of values that changes all the time based on the most recent effective round trip times between each two regions. Several factors influence this data:- 
- The current average latency of the underlying network connecting the two nodes which is based on the actual distances involved as well as the efficacy of the equipment that carries the traffic. As network links fail and traffic reroutes over backup links this value changes. 
- The handshake to set up (and tear down) a mutually secured transport layer connection two servers takes time even before latency makes it exponentially worse because it involves round-trip exchanges. Once a connection is extablished it can and should carry multiple remote procedure calls and notifications' data. The slow-start behaviopur of the TCP/IP networks we normally use acceptuate the desirability of reusing establoished connections. This setup time can be measured as the different between a round-trip time on an already established connection and the round-trip time when including setting up the connection. It's best to deduct observed round-trip latency from each individual measurement prior to comparison.
- But it's no good having a fast connection via a server or cluster that's so swamped with other work that the relaying of messages gets delayed. 

To monitor all these conditions and timings isn't particularly hard or resource intensive provided the strategy is for the monitoring to use existing traffic as much as possible and only add its own traffic on the rare occasion that there is no actual traffic from which to derive the mesaurements. It's already essential to monitor the regions and the links between them in case something fails which is almost certain to happen, so adding performance metrics on top of raw availability detection is cheap compared to the overall value it adds. Given the most recent set of metrics available including apparent outages they can be combined into a single weight or cost associated with every link between two nodes from the perspective of a specific region. With that cost a standard shortest path calculation yields the optimal path FeDist has to follow to carry information from one region to another at any given point in time. The only real impact is to firstly ensure that all the regions are on synchronised time and that the API protocol data units including the required timing data similar to what is used in NTP implementations.



















First among the reasons why the built-in provision for distributed processing in Erlang and Elixir (BEAM, to be exact) do not perform well outside of a LAN environment and even with a LAN but with more than a few hundred nodes in a cluster, is the default assumption it makes that all nodes are connected to all the other nodes in what is referred to as a full mesh. For the curious, this has two implications which adds to overheads as the number of nodes and/or the network latency increases. Firstly the resources required for each node to have and service an active network connection (socket) to each other node even if it never gets used. Secondly because the (visible) nodes emit heartbeats which the other nodes all monitor with the purpose of detecting when a node goes offline results in so much traffic and duplicate processing that it leaves neither network nor processing capacity open for actual workload purposes.  And that’s in a LAN environment where the heartbeats can be emitted using broadcast or multicast. Replace the LAN with a WAN and the traffic volumes that now results in effectively single-cast heartbeats very quickly overwhelms the network.




We want fully meshed regions, i.e. any region may ask any other region for data, actually keeping all those lines alive would eventually hog all our reources. The libary needs to know how the regions are connected and related but that information belongs to the application, not the libary, yet the application isn't involved enough in the actual communication to have accurate information about how well links perform - only the library knows that. If we combined the applicaiton and the functionality of the libary into one it would all be internal but each application would have to arrange for its own implementation of a suitable communication layer. To achieve the seperation of concerns we're hoping for, we require a way to exchange information between FeDist and applications such as Zyx. 

Fortunately Erlang implements several data structures which would be really useful for working with region data. 




Despite the 

By implication we need to avoid full mesh networking, but on a functional level each region must still be able to notify or fetch data from any other region.  So what we wish to achieve is to let the application call upon whatever region the data they’re loading points to and let the FeDist middleware determine how best to actually interconnect nodes either directly or via relay nodes to ensure the fastest turn-around times possible. 

