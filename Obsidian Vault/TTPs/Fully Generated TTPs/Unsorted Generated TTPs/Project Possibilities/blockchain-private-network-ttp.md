# Blockchain Private Network TTP

## Project Goal
The goal of this project is to set up a private blockchain network. By the end of this project, you will have implemented a functional blockchain network for secure, decentralized record-keeping or smart contract execution within a controlled environment.

## Overview
This TTP provides detailed instructions for implementing a private blockchain network using Hyperledger Fabric, a permissioned blockchain framework designed for enterprise use.

![Blockchain Network Overview](https://example.com/path/to/blockchain_network_overview.png)
*Figure 1: Blockchain Network Overview. Replace with an actual diagram showing the components of your private blockchain network.*

## Prerequisites
- Ubuntu 20.04 LTS or later
- Docker and Docker Compose installed
- Go programming language installed (version 1.14.x or later)
- Node.js and npm installed (version 12.x or later)
- Basic understanding of blockchain concepts
- Familiarity with command-line interfaces

## Step-by-step Guide

### 1. Install Prerequisites

Update the system and install necessary packages:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git curl wget software-properties-common -y
```

Install Docker and Docker Compose:

```bash
sudo apt install docker.io docker-compose -y
sudo usermod -aG docker $USER
```

Install Go:

```bash
wget https://golang.org/dl/go1.16.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.16.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
source ~/.bashrc
```

Install Node.js and npm:

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt install nodejs -y
```

### 2. Install Hyperledger Fabric Samples and Binaries

```bash
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.2 1.4.9
```

### 3. Navigate to the Fabric Samples Directory

```bash
cd fabric-samples/test-network
```

### 4. Start the Hyperledger Fabric Test Network

```bash
./network.sh up createChannel -c mychannel -ca
```

This command starts the network, creates a channel named "mychannel", and deploys the Certificate Authority.

![Fabric Network Startup](https://example.com/path/to/fabric_network_startup.png)
*Figure 2: Fabric Network Startup Output. Replace with an actual screenshot of the network startup process.*

### 5. Deploy a Chaincode (Smart Contract)

We'll deploy the asset-transfer (basic) chaincode:

```bash
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

### 6. Interact with the Network

Set up environment variables for the first organization:

```bash
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

Initialize the ledger with assets:

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
```

Query all assets:

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

![Chaincode Interaction](https://example.com/path/to/chaincode_interaction.png)
*Figure 3: Chaincode Interaction Output. Replace with an actual screenshot of the chaincode query output.*

### 7. Modify the Chaincode

1. Navigate to the chaincode directory:

```bash
cd ../asset-transfer-basic/chaincode-go
```

2. Open `chaincode/smartcontract.go` in a text editor and add a new function:

```go
func (s *SmartContract) UpdateAssetOwner(ctx contractapi.TransactionContextInterface, id string, newOwner string) error {
    asset, err := s.ReadAsset(ctx, id)
    if err != nil {
        return err
    }

    asset.Owner = newOwner

    assetJSON, err := json.Marshal(asset)
    if err != nil {
        return err
    }

    return ctx.GetStub().PutState(id, assetJSON)
}
```

3. Update the chaincode:

```bash
cd ../../test-network
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

### 8. Test the New Function

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"UpdateAssetOwner","Args":["asset1","NewOwner"]}'
```

### 9. Implement Access Control

1. Modify the `chaincode/smartcontract.go` file to include role-based access control:

```go
func (s *SmartContract) UpdateAssetOwner(ctx contractapi.TransactionContextInterface, id string, newOwner string) error {
    // Check if the client has the required role
    err := ctx.GetClientIdentity().AssertAttributeValue("role", "admin")
    if err != nil {
        return fmt.Errorf("submitting client not authorized to update asset owner")
    }

    // Rest of the function remains the same
    ...
}
```

2. Update the chaincode again using the `./network.sh deployCC` command.

### 10. Monitor the Network

1. View the Docker containers:

```bash
docker ps
```

2. Check the logs of a specific container:

```bash
docker logs <container_id>
```

3. Monitor the chaincode logs:

```bash
docker logs -f <chaincode_container_id>
```

![Docker Containers](https://example.com/path/to/docker_containers.png)
*Figure 4: Docker Containers Running the Blockchain Network. Replace with an actual screenshot of the docker ps output.*

## Advanced Topics

- Implementing complex smart contracts
- Scaling the network by adding new organizations
- Implementing private data collections
- Integrating with external systems using chaincode
- Implementing a custom consensus mechanism

## Troubleshooting

- If chaincode deployment fails, check the Docker logs for error messages
- For peer connection issues, verify the network configurations and certificates
- If transactions are failing, ensure that the client has the correct permissions and the chaincode is correctly installed on all required peers
- Use the Fabric CLI to diagnose network issues and query the ledger state

## Best Practices

1. Use version control for your chaincode and network configurations
2. Implement proper error handling and logging in your smart contracts
3. Regularly back up your ledger and network configurations
4. Implement proper access control and authentication mechanisms
5. Keep your Fabric components and dependencies up to date
6. Use separate networks for development, testing, and production

## Additional Resources

- [Hyperledger Fabric Documentation](https://hyperledger-fabric.readthedocs.io/)
- [Hyperledger Fabric Samples](https://github.com/hyperledger/fabric-samples)
- [Hyperledger Fabric Chaincode for Developers](https://hyperledger-fabric.readthedocs.io/en/release-2.2/chaincode4ade.html)
- [Blockchain for Business (Book)](https://www.oreilly.com/library/view/blockchain-for-business/9780135581353/)
- [Awesome Hyperledger Fabric (GitHub)](https://github.com/wearetheledger/awesome-hyperledger-fabric)

## Glossary

- **Blockchain**: A distributed ledger technology that maintains a growing list of records, called blocks, which are linked using cryptography
- **Smart Contract**: Self-executing contracts with the terms of the agreement directly written into code
- **Chaincode**: Hyperledger Fabric's term for smart contracts
- **Peer**: A node that commits transactions and maintains the state and a copy of the ledger
- **Orderer**: A node that orders transactions into a block
- **Channel**: A private subnet of communication between specific network members
- **Consensus**: The process of reaching agreement on the next set of transactions to be added to the ledger
- **MSP (Membership Service Provider)**: A component that defines the rules by which identities are validated, authenticated, and allowed access to a blockchain network

