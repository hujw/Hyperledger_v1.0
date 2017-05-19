# Hyperledger_v1.0

Hyperledger is a block-chain system developed by IBM. We use it to do our research in block-chain topic. The configuration files below are the important parts in this system.

  - The file "core.yaml" is used for "peer"
  
  - The file "orderer.yaml" is used for "orderer"
  
  - The file "configtx.yaml" defines how many organizations will join the channel. The system will use the information in this file to generate the channel (the tool "configtxgen").
  
  - The tool "cryptogen" will refer the file "crypto-config.yaml" to generate the "admincert", "keystone", and "cacert" for this demo. We can check the statement in "generateArtifacts.sh" for more detail information.
