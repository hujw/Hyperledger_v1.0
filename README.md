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
