# Subtreams Triggers - Importing transactions from the USDT Exchange Substreams.

# Table of Contents
- [Pre-requesites](#pre-requesites)
- [Launch local graph-node](#launch-local-graph-node)
- [Subgraph deployment](#subgraph-deployment)

## Pre-requesites 
Before proceeding, ensure you have all dependencies installed by running the following command:
```bash
yarn install 
```
## Launch local graph-node
To test your subgraph locally, make sure to deploy a local instance of the graph-node using the `./start.sh` script in `dev-environment` folder. 

Set your `SUBSTREAMS_API_TOKEN` and run the script in the `dev-environment` folder, by using the following commands: 

```bash
export SUBSTREAMS_API_TOKEN = "YOUR_TOKEN"
./start.sh -c # Flag -c can be added to clean the persistent folders prior running Postgres, IPFS and any similar required services
```
This script is running docker-compose to run all necessary instances to launch properly the node locally.


## Subgraph deployment
Now that your dev environment is fully set up, you are ready to deploy your subgraph!

1. Prepare the substreams

The [injective-common](https://substreams.dev/streamingfast/injective-common/v0.1.0) substreams has a `filtered_events` module that allows getting only the events that match certain types.

We will prepare a substreams package file (.spkg) with that parameter baked-in. Here, we want the event type `wasm`:

```yaml
# substreams.yaml
specVersion: v0.1.0
package:
  name: wasm_events
  version: v0.1.0

network: injective-mainnet
imports:
  injective: https://spkg.io/streamingfast/injective-common-v0.1.0.spkg

modules:
  - name: wasm_events
    use: injective:filtered_events

params:
  wasm_events: "wasm"
```

Using substreams v1.7.2 or above (https://github.com/streamingfast/substreams/releases), build the package:

```bash
substreams pack
```

This creates the `wasm-events-v0.1.0.spkg` file in the local folder, which will be referenced by subgraph.yaml.


2. Generate the code defined in the mappings

```bash
yarn codegen
```

3. Code your subgraph in `mappings.ts`, build it, deploy it on a local graph-node instance connected to injective-mainnet:

```bash
yarn build
yarn create-local
yarn run deploy-local
yarn run remove-local
```

4. Generate the Protobuf of the EventList that substreams outputs (and its dependencies):

```bash
# you probably want to delete the previous bindings
# rm -rf src/pb
buf generate --type="sf.substreams.cosmos.v1.EventList"  wasm-events-v0.1.0.spkg#format=bin
```

5. Publish to The Graph network:

```bash
yarn publish
```

