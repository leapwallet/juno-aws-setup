# Blockchain Node Setup

1. Install the toolchain:

    ```shell
    sudo apt -y install make build-essential gcc git jq chrony lz4
   
    #######################
    ## BEGIN: Install Go ##
    #######################
   
    wget https://golang.org/dl/go1.18.2.linux-amd64.tar.gz
    sudo tar xzvf go1.18.2.linux-amd64.tar.gz -C /usr/local
    rm go1.18.2.linux-amd64.tar.gz
   
    #####################
    ## END: Install Go ##
    #####################
   
    read -P 'A moniker is a name of your choosing for your node. Enter the moniker: ' MONIKER
   
    ##############################
    ## BEGIN: Select blockchain ##
    ############################## 
   
    set PROMPT '1 is for Commercio.network, 2 is for Juno, 3 is for Osmosis, 4 is for Sei, and 5 is for Stride. Enter'
    set PROMPT "$PROMPT which node you\re setting up: "
    read -P $PROMPT BLOCKCHAIN

    switch $BLOCKCHAIN
        case 1
            set DAEMON_NAME commercionetworkd
            set DAEMON_HOME ~/.commercionetwork
        case 2
            set DAEMON_NAME junod
            set DAEMON_HOME ~/.juno
        case 3
            set DAEMON_NAME osmosisd
            set DAEMON_HOME ~/.osmosisd
        case 4
            set DAEMON_NAME seid
            set DAEMON_HOME ~/.sei
        case 5
            set DAEMON_NAME strided
            set DAEMON_HOME ~/.stride
   
    ############################
    ## END: Select blockchain ##
    ############################ 
   
    ######################################
    ## BEGIN: Set environment variables ##
    ###################################### 
   
    printf "\
   
    set MONIKER $MONIKER
    
    # Go
    set -x GOROOT /usr/local/go
    set -x GOPATH ~/go
    set -x GO111MODULE on
    set PATH /usr/local/go/bin ~/go/bin $PATH

    # Cosmovisor
    set -x DAEMON_NAME $(printf $BLOCKCHAIN)d
    set -x DAEMON_HOME $DAEMON_HOME
    " >> ~/.config/fish/config.fish
   
    source ~/.config/fish/config.fish
      
    ####################################
    ## END: Set environment variables ##
    #################################### 
    ```
2. Set up the relevant blockchain node by picking the relevant link from the following list. If the blockchain node you're interested in isn't listed below, then pick a similar link anyway such as a validator setup link if you're setting up a validator, and follow similar instructions by referring to the blockchain node's official docs:
    - Commercio.network:
        - `testnet11k`:
            - [Validator](blockchain-nodes/commercio-network/testnet11k/validator.md)
    - Juno:
        - `juno-1`:
            - [Archive node](blockchain-nodes/juno/juno-1/archive-node.md)
    - Osmosis:
        - `osmo-test-4`:
            - [Archive node](blockchain-nodes/osmosis/osmo-test-4/validator.md)
    - Sei:
        - `atlantic-1`:
            - [Validator](blockchain-nodes/sei/atlantic-1/validator.md)
        - `atlantic-sub-1`:
            - [Validator](blockchain-nodes/sei/atlantic-sub-1/validator.md)
        - `atlantic-sub-2`:
            - [Validator](blockchain-nodes/sei/atlantic-sub-2/validator.md)
    - Stride:
        - `STRIDE-TESTNET-4`:
            - [Validator](blockchain-nodes/stride/stride-testnet-4/validator.md)
3. Enable Prometheus:

    ```shell
    sed 's|prometheus = .*|prometheus = true|' -i $DAEMON_HOME/config/config.toml
    sudo systemctl restart cosmovisor
    ```
4. Only follow this step on servers for validators or sentries. Configure:

    ```shell
    #########################
    ## BEGIN: Disable APIs ##
    #########################
   
    # The first expression disables the REST, gRPC, and gRPC Web APIs. The second enables the REST API for PANIC.
    sed \
        -e 's|enable = true|enable = false|' \
        -e '0,/enable = false/{s|enable = false|enable = true|}' \
        -i $DAEMON_HOME/config/app.toml
   
    #######################
    ## END: Disable APIs ##
    #######################
   
    ##################
    ## BEGIN: Prune ##
    ##################
   
    sed \
        -e 's|pruning = .*|pruning = "custom"|' \
        -e 's|pruning-keep-recent = .*|pruning-keep-recent = "100"|' \
        -e 's|pruning-keep-every = .*|pruning-keep-every = "0"|' \
        -e 's|pruning-interval = .*|pruning-interval = "10"|' \
        -i $DAEMON_HOME/config/app.toml
   
    sed 's|indexer = .*|indexer = "null"|' -i $DAEMON_HOME/config/config.toml
   
    ################
    ## END: Prune ##
    ################
      
    #############################################
    ## BEGIN: Improve performance and security ##
    #############################################
   
    sed \
        -e 's|max_num_inbound_peers = .*|max_num_inbound_peers = 100|' \
        -e 's|max_num_outbound_peers = .*|max_num_outbound_peers = 10|' \
        -e 's|flush_throttle_timeout = .*|flush_throttle_timeout = "100ms"|' \
        -i $DAEMON_HOME/config/config.toml
   
    ###########################################
    ## END: Improve performance and security ##
    ###########################################
   
    sudo systemctl restart cosmovisor
    ```