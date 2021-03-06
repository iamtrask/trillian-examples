# tether

#
Get a Trillian Log and Map running:

```bash
go get github.com/google/trillian
cd !$
go build ./server/trillian_log_server
go build ./server/trillian_log_signer
go build ./server/trillian_map_server

# In one terminal:
./trillian_log_server --logtostderr ...

# In another terminal:
./trillian_log_signer --logtostderr --force_master --http_endpoint=localhost:8092 --batch_size=1000 --sequencer_guard_window=0 --sequencer_interval=200ms

# In another terminal:
./trillian_map_server --logtostderr --rpc_endpoint=localhost:8095
```

Create a Log in Trillian:
```bash
go build ./cmd/createtree/
./createtree --admin_server=localhost:8090
<LOGID printed here>
```

Create a Map in Trillian:
```bash
go build ./cmd/createtree/
./createtree --admin_server=localhost:8095 --tree_type=MAP --hash_strategy=TEST_MAP_HASHER
<MAPID printed here>
```

Save the LOGID and MAPID, you will need them later.

Build and run geth.
We're going to use the rinkeby.io test-net because everything takes too long on
the main net :)


```bash
# In yet another terminal:
go get github.com/ethereum/go-ethereum
cd ${GOPATH}/src/github.com/ethereum/go-ethereum
make geth

# download rinkeby config, and init the data dir:
wget https://www.rinkeby.io/rinkeby.json
build/bin/geth --datadir=$HOME/rinkeby init rinkeby.json

# Finally, run geth to sync the data:
build/bin/geth --networkid=4 --datadir=$HOME/rinkeby --cache=1024 --syncmode=full --verbosity 3 --ethstats='yournode:Respect my authoritah!@stats.rinkeby.io' --bootnodes=enode://a24ac7c5484ef4ed0c5eb2d36620ba4e4aa13b8c84684e1b4aab0cebea2ae45cb4d375b77eab56516d34bfbd3c1a833fc51296ff084b770b94fb9028c4d25ccf@52.169.42.101:30303 --rpc console

```

Build and run the tether Follower (here's where you use your saved LOGID):

```bash
# Yes, another terminal:
go get github.com/google/trillian-examples
cd $(GOPATH)/github.com/google/trillian-examples
go run ./etherslurp/cmd/follower/main.go --geth=http://127.0.0.1:8545 --trillian_log=localhost:8090 --log_id LOGID --logtostderr
```

Build and run the tether Mapper (here's where you use your saved LOGID and MAPID):

```bash
# ... yup
cd $(GOPATH)/github.com/google/trillian-examples
go run ./etherslurp/cmd/mapper/main.go --logtostderr --trillian_log=localhost:8090 --log_id LOGID --map_id MAPID
```

Watch as your diskspace gets eaten.

# UI
If you'd like to inspect the contents of the map, there's a very simple web UI you can use to do so (again, using your saved MAPID):

```bash
# in another terminal, run:
cd $(GOPATH)/github.com/google/trillian-examples
go run ./etherslurp/cmd/ui/main.go --logtostderr --trillian_map=localhost:8095 --map_id=MAPID
```
