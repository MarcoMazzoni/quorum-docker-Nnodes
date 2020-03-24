# quorum-docker-Nnodes

## Modified by me
  * Add support for Tessera (replacing Constellation).
  * Fixed bugs in setup.sh for Istanbul BFT.
  * Added option for deterministic Ethereum account generation (for testing purpose).

## Modified by @pccr10001
  * Add support for Quorum 2.3.0.
  * Add support for Clique and IBFT consensus engine.
  * Add support for Multi-Cluster.

## Intro

Run a bunch of Quorum nodes, each in a separate Docker container.

This is simply a learning exercise for configuring Quorum networks. Probably best not used in a production environment.

In progress:

  * Add multi-nodes deployment.
  * Further work on Docker image size.
  * Tidy the whole thing up.

See the *README* in the *Nnodes* directory for details of the set up process.

## Building

In the top level directory:

    docker build -t quorum .
    
The first time will take a while, but after some caching it gets much quicker for any minor updates.

## Running

Change to the *Nnodes/* directory. Edit the `ips` variable in *config.sh* to list two or more IP addresses on the Docker network that will host nodes:

    # change to Nnodes
    cd Nnodes
    
    vi config.sh
    
    # Total nodes to deploy
    total_nodes=5

    # Signer nodes for Clique and IBFT
    signer_nodes=7
    
    # auto start
    auto_start_containers=true
    
The IP addresses are needed for Tessera to work. Now run,

    ./setup.sh [raft]
    ./setup.sh clique    # For Clique
    ./setup.sh istanbul  # For IBFT
    
This will set up as many Quorum nodes as IP addresses you supplied, each in a separate container, on a Docker network, all hopefully talking to each other.

    Nnodes> docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
    83ad1de7eea6        quorum              "/qdata/start-node.sh"   55 seconds ago      Up 53 seconds       0.0.0.0:22002->8545/tcp   nnodes_node_2_1
    14b903ca465c        quorum              "/qdata/start-node.sh"   55 seconds ago      Up 54 seconds       0.0.0.0:22003->8545/tcp   nnodes_node_3_1
    d60bcf0b8a4f        quorum              "/qdata/start-node.sh"   55 seconds ago      Up 54 seconds       0.0.0.0:22001->8545/tcp   nnodes_node_1_1

## Stopping

    docker-compose down
  
## Playing

### Accessing the Geth console

To enter Geth console, use:

    ./cmd.sh console 1

Or you have Geth installed on the host machine you can do the following from the *Nnodes* directory to attach to Node 1's console.

    geth attach qdata_1/dd/geth.ipc

### View Geth logs

To show Geth log:

    ./cmd.sh log 1

To show all Geth node logs:

    ./cmd.sh logs


## Notes

The RPC port for each container is mapped to *localhost* starting from port 22001. So, to see the peers connected to Node 2, you can do either of the following and get the same result. Change it in *setup.sh* if you don't like it.

    curl -X POST --data '{"jsonrpc":"2.0","method":"admin_peers","id":1}' 172.13.0.3:8545
    curl -X POST --data '{"jsonrpc":"2.0","method":"admin_peers","id":1}' localhost:22002

You can see the log files for the nodes in *qdata_N/logs/geth.log* and *qdata_N/logs/tessera.log*.  This is useful when things go wrong!

This example uses only the Raft consensus mechanism.

