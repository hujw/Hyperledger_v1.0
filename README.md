# Hyperledger_v1.0

Hyperledger is a block-chain system developed by IBM. We use it to do our research in block-chain topic. The configuration files below are the important parts in this system.

  - The file "core.yaml" is used for "peer" (placed in "crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/")
  
  - The file "orderer.yaml" is used for "orderer" 
  
  - The file "configtx.yaml" defines how many organizations will join the channel. The system will use the information in this file to generate the channel (the tool "configtxgen").
  
  - The tool "cryptogen" will refer the file "crypto-config.yaml" to generate the "admincert", "keystone", and "cacert" for this demo. We can check the statement in "generateArtifacts.sh" for more detail information.

The configuration in Hyperledger will follow the defined order in the core "core/config/config.go" 
```sh
func InitViper(v *viper.Viper, configName string) error {
	var altPath = os.Getenv("FABRIC_CFG_PATH")
	if altPath != "" {
		// If the user has overridden the path with an envvar, its the only path
		// we will consider
		addConfigPath(v, altPath)
	} else {
		// If we get here, we should use the default paths in priority order:
		//
		// *) CWD
		// *) The $GOPATH based development tree
		// *) /etc/hyperledger/fabric
		//

		// CWD
		addConfigPath(v, "./")

		// DevConfigPath
		err := AddDevConfigPath(v)
		if err != nil {
			return err
		}

		// And finally, the official path
		if dirExists(OfficialPath) {
			addConfigPath(v, OfficialPath)
		}
	}

	// Now set the configuration file.
	if v != nil {
		v.SetConfigName(configName)
	} else {
		viper.SetConfigName(configName)
	}

	return nil
}
```

Now, we assume there are two servers, one (named S1) is acted as Orderer as well as Org1's Peer0 and the other (named S2) is acted as Org2's Peer0. The commands we execute for this demo.

```sh
[For Orderer]
$cd src/github.com/hyperledger/fabric/examples/e2e_cli
$./generateArtifacts.sh nchc
$../../build/bin/orderer
```
```sh
[For Peer0 in Org1]
$cd src/github.com/hyperledger/fabric/examples/e2e_cli
$export FABRIC_CFG_PATH=crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/
$../../build/bin/peer node start --peer-defaultchain=false
```
In S2, use command "scp" to copy the directory "crypto-config" created by "cryptogen" from S1.
```sh
[For Peer0 in Org2]
$cd src/github.com/hyperledger/fabric/examples/e2e_cli
$export FABRIC_CFG_PATH=crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/
```
Modify core.yaml and put it in $FABRIC_CFG_PATH.
```sh
$../../build/bin/peer node start --peer-defaultchain=false
```
Finally, we use another terminal and setup the environment for "cli". Congratulation! we can create and join channel, moreover, install and invoke the chaincode.
```sh
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_TLS_ROOTCERT_FILE=/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export ORDERER_CA=/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem
```
```sh
$export PATH=$PATH:$PWD/../../build/bin
$peer channel create -o orderer.example.com:7050 -c nchc -f channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA
$peer channel join -b nchc.block
$peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
$peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C nchc -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR ('Org0MSP.member','Org1MSP.member')" 
```
Finnally, we can use "docker ps" to check the result.
```sh
$ docker ps
CONTAINER ID   IMAGE                                 COMMAND                  CREATED             STATUS       
8cd54ba7ec0b   dev-peer0.org1.example.com-mycc-1.0   "chaincode -peer.a..."   13 seconds ago      Up 12 seconds                         

$ docker logs -f dev-peer0.org1.example.com-mycc-1.0
ex02 Init
Aval = 100, Bval = 200
```
Query for the value of "a".
```sh
$ peer chaincode query -C nchc -n mycc -c '{"Args":["query","a"]}'
2017-05-19 14:21:14.958 CST [msp] getMspConfig -> INFO 001 intermediate certs folder not found at [/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/intermediatecerts]. Skipping.: [stat /home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/intermediatecerts: no such file or directory]
2017-05-19 14:21:14.958 CST [msp] getMspConfig -> INFO 002 crls folder not found at [/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/intermediatecerts]. Skipping.: [stat /home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/crls: no such file or directory]
2017-05-19 14:21:14.958 CST [msp] getMspConfig -> INFO 003 MSP configuration file not found at [/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/config.yaml]: [stat /home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/config.yaml: no such file or directory]
2017-05-19 14:21:15.000 CST [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
2017-05-19 14:21:15.001 CST [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
2017-05-19 14:21:15.001 CST [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AA6070A6008031A0A08DB9DFAC80510...6D7963631A0A0A0571756572790A0161
2017-05-19 14:21:15.001 CST [msp/identity] Sign -> DEBU 007 Sign: digest: D2D4E8E8EC3A083ADCC7B63257692D355FBC43F2020E9608BB8436533EDB475C
Query Result: 100
2017-05-19 14:21:15.009 CST [main] main -> INFO 008 Exiting.....
```
Invoke the transaction for moving "10" from "a" to "b".
```sh
$peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C nchc -n mycc -c '{"Args":["invoke","a","b","10"]}'
2017-05-19 14:25:34.956 CST [msp] getMspConfig -> INFO 001 intermediate certs folder not found at [/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/intermediatecerts]. Skipping.: [stat /home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/intermediatecerts: no such file or directory]
2017-05-19 14:25:34.956 CST [msp] getMspConfig -> INFO 002 crls folder not found at [/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/intermediatecerts]. Skipping.: [stat /home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/crls: no such file or directory]
2017-05-19 14:25:34.956 CST [msp] getMspConfig -> INFO 003 MSP configuration file not found at [/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/config.yaml]: [stat /home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/config.yaml: no such file or directory]
2017-05-19 14:25:34.999 CST [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
2017-05-19 14:25:34.999 CST [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
2017-05-19 14:25:35.001 CST [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AA6070A6008031A0A08DF9FFAC80510...696E766F6B650A01610A01620A023130
2017-05-19 14:25:35.001 CST [msp/identity] Sign -> DEBU 007 Sign: digest: E55EAC52DA04739C062F9D4FDB918B59E541D1A16A78879AB950A19590052173
2017-05-19 14:25:35.011 CST [msp/identity] Sign -> DEBU 008 Sign: plaintext: 0AA6070A6008031A0A08DF9FFAC80510...905566A8D4C51DAE53CA9DE94D462DA0
2017-05-19 14:25:35.011 CST [msp/identity] Sign -> DEBU 009 Sign: digest: C634036F66F008253AF3E376778AD3A15583932BA8A9828612145DF1E4C4128D
2017-05-19 14:25:35.015 CST [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 00a Invoke result: version:1 response:<status:200 message:"OK" > payload:"\n \310\237\311\2209Z_HPjP\311\010\303\226\003\321\355\330z qZ\353$\270^\014\307=\301\367\022Y\nE\022\024\n\004lscc\022\014\n\n\n\004mycc\022\002\010\001\022-\n\004mycc\022%\n\007\n\001a\022\002\010\001\n\007\n\001b\022\002\010\001\032\007\n\001a\032\00290\032\010\n\001b\032\003210\032\003\010\310\001\"\013\022\004mycc\032\0031.0" endorsement:<endorser:"\n\007Org1MSP\022\325\006-----BEGIN -----\nMIICWjCCAgCgAwIBAgIQKnSFqt9Vhgg0ZpAwVs+K6TAKBggqhkjOPQQDAjBzMQsw\nCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNU2FuIEZy\nYW5jaXNjbzEZMBcGA1UEChMQb3JnMS5leGFtcGxlLmNvbTEcMBoGA1UEAxMTY2Eu\nb3JnMS5leGFtcGxlLmNvbTAeFw0xNzA1MTkwNTE1MDlaFw0yNzA1MTcwNTE1MDla\nMFsxCzAJBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1T\nYW4gRnJhbmNpc2NvMR8wHQYDVQQDExZwZWVyMC5vcmcxLmV4YW1wbGUuY29tMFkw\nEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEjgpKheH5x0v6od8Uns4NLQX5qLsMH8FP\nDkeX3Tl9liA//jpzQ7vvlNKPQEj7PrdqZK9MVFBWyFCTi0OYNEYc8aOBjTCBijAO\nBgNVHQ8BAf8EBAMCBaAwEwYDVR0lBAwwCgYIKwYBBQUHAwEwDAYDVR0TAQH/BAIw\nADArBgNVHSMEJDAigCAQ8I5mNIcn3Optv7QpGuLdjgmUISwluEcSFA/ZN9HJnzAo\nBgNVHREEITAfghZwZWVyMC5vcmcxLmV4YW1wbGUuY29tggVwZWVyMDAKBggqhkjO\nPQQDAgNIADBFAiEAg46jfbvnOTKQLv148E1cAs+qH/cxZKka97+jPw7gYggCIHKz\naX38u1dorc2gPTcj5XvScMO9ivHY75N+xRNeb34p\n-----END -----\n" signature:"0D\002 ~\0064\207\361\336_\004\027\227\323\r\327$\337v\177\371\326\273[\004\223\023\256.}\217I\034\333a\002 \t\215\210\203\240y\027t\356\004\250\247\001I\345\273\220Uf\250\324\305\035\256S\312\235\351MF-\240" >
2017-05-19 14:25:35.015 CST [main] main -> INFO 00b Exiting.....
```
Query the value of "a" again! The value is changed to "90" (the initial value is 100).
```sh
$ peer chaincode query -C nchc -n mycc -c '{"Args":["query","a"]}'
2017-05-19 14:25:55.644 CST [msp] getMspConfig -> INFO 001 intermediate certs folder not found at [/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/intermediatecerts]. Skipping.: [stat /home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/intermediatecerts: no such file or directory]
2017-05-19 14:25:55.644 CST [msp] getMspConfig -> INFO 002 crls folder not found at [/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/intermediatecerts]. Skipping.: [stat /home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/crls: no such file or directory]
2017-05-19 14:25:55.644 CST [msp] getMspConfig -> INFO 003 MSP configuration file not found at [/home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/config.yaml]: [stat /home/jwhu/work/src/github.com/hyperledger/fabric/examples/e2e_cli/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/config.yaml: no such file or directory]
2017-05-19 14:25:55.687 CST [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
2017-05-19 14:25:55.687 CST [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
2017-05-19 14:25:55.688 CST [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AA8070A6208031A0C08F39FFAC80510...6D7963631A0A0A0571756572790A0161
2017-05-19 14:25:55.688 CST [msp/identity] Sign -> DEBU 007 Sign: digest: 5B71A160C40F989671D2303639A73D58BF3EA0BF981C13CB54A09AFA38AF93F1
Query Result: 90
2017-05-19 14:25:55.695 CST [main] main -> INFO 008 Exiting.....
```
