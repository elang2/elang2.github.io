This is the second part of the Cassandra series and I will try to talk about Scaling Strategy.


In this first part I will focus on the <b> setup </b> of <a href="http://cassandra.apache.org/">Cassandra</a> on AWS.


#Deployment Model


Cassandra will be set up as a multi-node cluster on AWS. AWS is
spread over multiple regions and each region has multiple availability
zones(AZ) within them.

Each region will start with at-least 3 nodes in a cluster. The nodes will be spread across different AZ's. Most of the AWS regions have atleast 3 different AZ.

In regions where there are less than 3 AZ's, two nodes will share the same AZ.

<figure class="half center">
<img src="/images/cassandra-aws/cassandra-aws-archi.png" height="400px"></img>

<figcaption>Fig 1: Cassandra deployment in a region spread over multiple availability zones (AZ)</figcaption>
</figure>



#Setup


##AWS 

- Choose a region which has three availability zones. For the purpose of this post, I have choosen <b>us-west-2</b>.
- Spin up one EC2 instance in each availability zone.
- Databased should always be on private subnets of VPC and should not be open to internet.
- Update the firewall setting on the secutiry group associated with the EC2 instance to allow incomming traffinc on the following ports:

	9042	CQL Client port
	7000	Inter node communication on cluster

##Installable 

- Download Cassandra 2.2.4V from the below URL.

   http://downloads.datastax.com/community/dsc-cassandra-2.2.4-bin.tar.gz

- Unzip it to a location on the EC2 machine. 
- Do this on all three nodes.


##Cassandra Configuration
 
Make the below changes in the files present in the `/conf` folder of Cassandra.



`File : cassandra-rackdc.properties`

{% highlight yaml %}

dc_suffix = 2a_cassandra   # Uniquely identify a node in a datacenter(DC).DC names are automatically assiged by Cassandra using EC2Snitch/EC2MultiRegionSnitch.
prefer_local = true

{% endhighlight %}




`File : cassandra.yaml`

{% highlight yaml %}
partitioner: org.apache.cassandra.dht.Murmur3Partitioner  # This is the default and we will keep it as it is. Used to hash and distribute the keys across differnt nodes.

endpoint_snitch: Ec2Snitch  # Use EC2MultiRegionSnitch if you are setting up multiple clusters spanning different regions. Otherwise use Ec2Snitch

listen_address: 10.101.212.201  # This will be the private IP address of the EC2 instance. Will vary from instance to instance.

broadcast_address: 10.101.212.201  # private IP address of the EC2 instance.

rpc_address: 10.101.212.201  # private IP address of the EC2 instance

seeds: "10.101.212.206,10.101.214.60"   ## IP address of the nodes acting as seeds

key_cache_size_in_mb: 100000

data_file_directories:   # location where database files needs to be stored. 
   -/local/mnt/cassandra/data
   
commitlog_directory: /local/mnt/cassandra/commitlog  #location where the commit logs needs to be stored.

{% endhighlight %}



*Cassandra Client for Java *

Recommended way to interface with Cassandra is through the datastax
Java driver. This driver leverages CLQ to talk to the Cassandra
instance.

< dependency>

< groupId>com.datastax.cassandra< /groupId>

< artifactId>cassandra-driver-core< /artifactId>

< version>2.1.6< /version>

< /dependency>

For Spring applications the database abstraction for Cassandra is provided via Spring-Data-Cassandra library.

< dependency>

< groupId>org.springframework.data< /groupId>

< artifactId>spring-data-cassandra< /artifactId>

< /dependency>
Cassandra Scaling Strategy

Cassandra scaling will be based on two factors :

    Latency of response for a request.
    Amount of disk space left on the Cassandra node.

The cluster will always scale up and never down. Scaling up will
improve the latency of a request and also provide new disk space to
provide new incoming data.

Scaling will be done as at a cluster level as follows :

Strategy 1

    Select the cluster to scale.
    Select AZ's with low number of nodes.
    Choose two AZ's and add one node to each of the AZ's.

Strategy 2

    A cluster will always have a minimum of 3 nodes. Hence scaling in a cluster will be achieve in increments of 3 nodes.
    If a region has 3 AZ's then add one node to each AZ.
    If a region has less than 3 AZ's then one of the AZ will end up having more nodes than the other AZ.

Scaling Trigger Points or Threshold

    -Latency > 500ms for more than 1 minute.
    -Scale when disk space remaining is less than 30% of the total available diskspace on the node.

 

Caching Configuration

 

    Example:
    CREATE TABLE USER(
    user_id uuid,
    email_id text,
    password text,
    oem_id UUID,
    first_name text,
    last_name text,
    work_phone text,
    work_phone_code text,
    mobile_phone text,
    mobile_phone_code text,
    status text,
    primary_user boolean,
    created_by text,
    created_time timestamp,
    updated_by text,
    updated_time timestamp,
    user_roles set,
    PRIMARY KEY(email_id)) WITH caching = { 'keys' : 'ALL', 'rows_per_partition' : '200' };

 

    The above query will create a table with row caching enabled. The
    first 200 records in the partition will be cached. This means that
    Cassandra will first look into the cache for the data and then look into
    the SSTable if the data is not found in the cache.

    Use row caching only for those scenarios where very low latency is required.

Cassandra Data Backup Strategy

In case of a node failure in a cluster, Cassandra can automatically
repair and get the node up to speed on the data using the replicated
data that is present on the other nodes.

It is advised to enabled incremental backup and then collect snapshots at a regular intervals( say 24 hrs).

The snapshot backups and incremental backups can be collected from the nodes and then pushed into S3 for use during restore.

        Snapshot backup should be taken on a daily basis at 12:00 A.M midnight.
        Snapshots & incremental backup files older than 7 days can be deleted from S3.
        S3 backup location should be named and organized based on Cluster and node names. In case of AWS these will be Region and AZs.

Reference

[Incremental Backup]

[Taking a Snapshot]

[Restoring from a snapshot]
Cassandra Node Failure Recovery Strategy

If a node in a cluster fails or if the node restarts and is out of
the cluster for a certain time then it is mandatory to run the node
repair tool on the node.

Execute the following command on the failed node soon after startup:

"nodetool repair -dc < datacenter or AZ name> -h localhost

Example:
"nodetool repair -dc us-west-2 -h localhost

Node repair makes sure that that old delete data(Tombstoned) does not resurrect as new data on this node.
Cassandra Unit Testing

Cassandra Unit provides a library which sprins off an embedded Cassandra server for testing against.

This is very useful for performing Unit/Integration testing of code related to Cassandra.

Steps for Spring JUnit :

    Include the following dependencies in POM

< dependency>

     < groupId>org.cassandraunit< /groupId>
     < artifactId>cassandra-unit-spring< /artifactId>
     < version>2.1.3.1</version>
     < scope>test< /scope>

< /dependency>

< dependency>

     < groupId>org.cassandraunit< /groupId>
     < artifactId>cassandra-unit< /artifactId>
     < version>2.0.2.1< /version>

< /dependency>

2 . Annotate your Junit class with the following

@SpringApplicationConfiguration(classes = CassandraConfiguration.class)

@TestExecutionListeners({ CassandraUnitDependencyInjectionTestExecutionListener.class,
DependencyInjectionTestExecutionListener.class })

@CassandraDataSet(value = { "create-table.cql" }, keyspace = "ioe")

@EmbeddedCassandra

@Configuration

@RunWith(SpringJUnit4ClassRunner.class)

3 . Place your table creating queries in a file named 'create-table.cql' under /resource folder of your TEST .

Cassandara Performance Testing[ Work in Progress]

Leverage the Jmeter plugin for Cassandra developed by Netflix.

[CassJMeter]
 