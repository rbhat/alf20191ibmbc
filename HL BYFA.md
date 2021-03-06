<img src="https://farm5.staticflickr.com/4503/37148677233_71edc5a37b_o.png" width="1041" height="53" alt="blueband">


<img src="/Screen Shot 2018-12-30 at 08.48.23.png">

# [Build your first Hyperedger Fabric Network](HL%20BYFA.md)

[Documentation](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html#building-your-first-network)

## Step 1: Make sure you have the right pre-reqs.

So let's start with <a href="https://hyperledger-fabric.readthedocs.io/en/release-1.4/prereqs.html#prerequisites"> the pre-reqs</a> 

## Step 2 Download the Docker images of the Fabric code from GitHub

We will also need to install <a href="https://hyperledger-fabric.readthedocs.io/en/release-1.4/install.html#install-samples-binaries-and-docker-images">Binaries and Docker Images with curl</a>, see line below:

~~~~
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.4.0  
~~~~

creates the directory fabric-samples and pulls down the following docker images:

~~~~

hyperledger/fabric-tools                                                                                             
hyperledger/fabric-ccenv                                                                                                 hyperledger/fabric-orderer                                                                            
hyperledger/fabric-peer                                                                                              
hyperledger/fabric-ca                                                                                                       hyperledger/fabric-javaenv                                                                               
hyperledger/fabric-zookeeper                                                                             
hyperledger/fabric-kafka                                                                                              
hyperledger/fabric-couchdb                                                                                            
hyperledger/fabric-baseimage                                                                                     
hyperledger/fabric-tools                                                                                                      
~~~~

Hyperledger is advancing quite quickly so the version number may well have changed by the time you see this article.
The download will take a minute or two. The download will install a directory called fabric-samples. Take time to familiarize yourself with the Hyperledger docker images that you have downloaded.

## Step 3 Let's build our first network with the byfn.sh script.

We are now ready to [build our first network](https://hyperledger-fabric.readthedocs.io/en/release-1.4/build_network.html) 

cd fabric-samples/first-network
./byfn.sh generate

Which does the following:

1. Generate certificates using cryptogen tool 
1. Generating Orderer Genesis block (The first block of the blockchain)
1. Generating channel configuration transaction 'channel.tx'
1. Generating anchor peer update for Org1MSP  (MSP = Member Service Provider)
1. Generating anchor peer update for Org2MSP 

Followed by running <b>./byfn.sh up -l node</b> which brings up the Hyperledger network on your laptop. 

~~~~

Build your first network (BYFN) end-to-end test
Channel name : mychannel
Creating channel...
Endorser and orderer connections initialized
===================== Channel 'mychannel' created ===================== 
Having all peers join the channel...
peer0.org1 joined channel 'mychannel'
peer1.org1 joined channel 'mychannel'
peer0.org2 joined channel 'mychannel'
peer1.org2 joined channel 'mychannel'
Anchor peers updated for org 'Org1MSP' on channel 'mychannel'
Anchor peers updated for org 'Org2MSP' on channel 'mychannel'
Chaincode is installed on peer0.org1
peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/
Chaincode is installed on peer0.org2 
Chaincode is instantiated on peer0.org2 on channel 'mychannel'
Querying on peer0.org1 on channel 'mychannel'...
Query successful on peer0.org1 on channel 'mychannel'
Invoke transaction successful on peer0.org1 peer0.org2 on channel 'mychannel'
Querying on peer1.org2 on channel 'mychannel'...
Query successful on peer1.org2 on channel 'mychannel' =
All GOOD, BYFN execution completed
~~~~

The chaincode used is /fabric-samples/chaincode/chaincode_example02/node


## Step 4 At the end of the run we bring down the network 

~~~~

./byfn.sh down

Stopping cli                    ... done
Stopping peer0.org1.example.com ... done
Stopping orderer.example.com    ... done
Stopping peer1.org1.example.com ... done
Stopping peer1.org2.example.com ... done
Stopping peer0.org2.example.com ... done
Removing orphan container "couchdb"
Removing orphan container "ca.example.com"
Removing cli                    ... done
Removing peer0.org1.example.com ... done
Removing orderer.example.com    ... done
Removing peer1.org1.example.com ... done
Removing peer1.org2.example.com ... done
Removing peer0.org2.example.com ... done
Removing network net_byfn

~~~~

Let's look at the Chaincode used: Arnes-MBP: chaincode_example02.js 

~~~~
/*
Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
*/

const shim = require('fabric-shim');
const util = require('util');

var Chaincode = class {

  // Initialize the chaincode
  async Init(stub) {
    console.info('========= example02 Init =========');
    let ret = stub.getFunctionAndParameters();
    console.info(ret);
    let args = ret.params;
    // initialise only if 4 parameters passed.
    if (args.length != 4) {
      return shim.error('Incorrect number of arguments. Expecting 4');
    }

    let A = args[0];
    let B = args[2];
    let Aval = args[1];
    let Bval = parseInt(Bvalbytes.toString());
    // Perform the execution
    let amount = parseInt(args[2]);
    if (typeof amount !== 'number') {
      throw new Error('Expecting integer value for amount to be transaferred');
    }

    Aval = Aval - amount;
    Bval = Bval + amount;
    console.info(util.format('Aval = %d, Bval = %d\n', Aval, Bval));

    // Write the states back to the ledger
    await stub.putState(A, Buffer.from(Aval.toString()));
    await stub.putState(B, Buffer.from(Bval.toString()));

  }

  // Deletes an entity from state
  async delete(stub, args) {
    if (args.length != 1) {
      throw new Error('Incorrect number of arguments. Expecting 1');
    }

    let A = args[0];

    // Delete the key from the state in ledger
    await stub.deleteState(A);
  }

  // query callback representing the query of a chaincode
  async query(stub, args) {
    if (args.length != 1) {
      throw new Error('Incorrect number of arguments. Expecting name of the person to query')
    }

    let jsonResp = {};
    let A = args[0];

    // Get the state from the ledger
    let Avalbytes = await stub.getState(A);
    if (!Avalbytes) {
      jsonResp.error = 'Failed to get state for ' + A;
      throw new Error(JSON.stringify(jsonResp));
    }

    jsonResp.name = A;
    jsonResp.amount = Avalbytes.toString();
    console.info('Query Response:');
    console.info(jsonResp);
    return Avalbytes;
  }
};

shim.start(new Chaincode());
 
~~~~

