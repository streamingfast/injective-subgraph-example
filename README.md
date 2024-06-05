# Template for Substreams-powered-Subgraphs on Injective

This repository is a template to create, deploy an `injective subgraph` based on `substreams`. 

By going through it, you will learn how to create your own subgraph taking his data source from  `substreams fundational module` on `injective`. 
You will also learn how to deploy your subgraph on a `local` graph-node instance or deploy it to an external node and publish it through `the Graph Network`.

# Table of Contents
- [Pre-requesites](#pre-requesites)
  - [Install Substreams](#install-substreams)
  - [Install Dependencies](#install-dependencies)
  - [Install Docker (Optional)](#install-docker-optional)
  - [Get Substreams API Token (Optional)](#get-substreams-api-token-optional)
- [Launch local graph-node (Optional)](#launch-local-graph-node-optional)
- [Write a Subgraph](#write-a-subgraph)
  - [Prepare the substreams](#prepare-the-substreams)
  - [Generate schema entities](#generate-schema-entities)
  - [Generate additional Protobufs (Optional)](#generate-additional-protobufs-optional)
- [Deploy a subgraph](#deploy-a-subgraph)
  - [Deploy on local graph-node (Optional)](#deploy-on-local-graph-node-optional)
  - [Deploy on Subgraph studio](#deploy-on-subgraph-studio)
- [Publish a Subgraph](#publish-a-subgraph)
- [Local Graph Node Management](#local-graph-node-management-optional)
  - [Reset node](#reset-node)
  - [Check logs](#check-logs)

## Pre-requesites

### Install substreams
Before starting, you need to install the `substreams CLI` (v1.7.2 or above). You got many options as explained in this [installation guide](https://substreams.streamingfast.io/documentation/consume/installing-the-cli). 
One way to do it is to use Homebrew:

```bash
# Installation via Homebrew
brew install streamingfast/tap/substreams
```

To check if `substreams` installation was successful, you can run the following command:

```bash
substreams version 
```

### Install Dependencies
The next step is to install the all necessary dependencies using `yarn` or `npm` package manager.
Run the following command in the `root` of the repository:

```bash
yarn install
```
or 
```bash
npm install
```

### Get Substreams API Token (Optional)
If you intend to run a local graph-node instance, you will need to get a substreams API token. You local graph-node instance needs to connect to the substreams API to get source data.
To interact with the substreams API, it needs to set an API token. You can get it by following the instructions on the [authentification section](https://substreams.streamingfast.io/documentation/consume/authentication) in the `StreamingFast` documentation.

### Install Docker (Optional)
If you intend to run a local graph-node instance, you will need to install Docker. You can do it by following the instructions on the [official Docker website](https://docs.docker.com/get-docker/).

## Launch local graph-node (Optional)
To deploy your subgraph locally, you need to run a local graph-node instance. To do so, export your `SUBSTREAMS_API_TOKEN` and
use the `launch-graph-node` script :

```bash
export SUBSTREAMS_API_TOKEN = "YOUR_TOKEN"
yarn launch-graph-node 
```
This script is running docker-compose to create all necessary instances to launch properly the node locally on the `injective-mainnet` network.

## Write a Subgraph 

### Prepare the substreams

When writing a `substreams powered subgraph`, your subgraph is using substreams as a source of data. 
In this template, we are using the [injective-common](https://substreams.dev/streamingfast/injective-common/v0.1.0) substreams has a `filtered_events` module 
that allows getting only the events matching certain types.

To use your substreams as a source, you must first pack it into a package file (.spkg). In the template example, we are using a substreams module that filters events related to `wasm_events` parameter. 
In this case, we are filtering events related to `wasm` events.

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
Note: You can edit the `substreams.yaml` and choose a desired name for events to filter

Once your `substreams` manifest (substreams.yaml) is ready, you can pack it into a package file (.spkg):

```bash
substreams pack
```
Note: To do so, you will need substreams v1.7.2 or above

This creates the `wasm-events-v0.1.0.spkg` file in the local folder, which will be referenced by subgraph.yaml.

### Generate schema entities 

* Subgraphs write data using a structure known as an entity.
* Define them in your [schema.graphql](./schema.graphql) according to this documentation: https://thegraph.com/docs/en/developing/creating-a-subgraph/#the-graphql-schema
* Then, generate the `AssemblyScript` types:

```bash
yarn codegen
```

### Generate additional protobufs (Optional) 

Your subgraph may use protobuf generation depending on which substreams module you got in source. In the template subgraph, the `substreams source module` has the `sf.substreams.cosmos.v1.EventList` protobuf type as output. Protobufs used in the `substreams source module` must be contained within the `substreams package` previously build. To generate needed protobufs, you can use the following command.

```bash
# you probably want to delete the previous bindings
# rm -rf src/pb
buf generate --type="sf.substreams.cosmos.v1.EventList"  wasm-events-v0.1.0.spkg#format=bin
```
Note: In our example, we generate the `sf.substreams.cosmos.v1.EventList` contains within the `wasm-events-v0.1.0.spkg` package.

Now that your dev environment is fully set up, you are ready to deploy your subgraph!

## Deploy a subgraph
Once you have finished writing your subgraph, you can now build, create and deploy it through your `local graph-node` instance or in `Subgraph studio`.

### Deploy on local graph-node (Optional)
To deploy it on your `local graph-node` intance:

```bash
yarn build
yarn create-local
yarn deploy-local
#yarn remove-local 
```
Note: The `yarn remove-local` command is used to remove the subgraph from the local graph-node instance.

### Deploy on Subgraph studio
To deploy your subgraph in `Subgraph Studio`:

```bash
yarn build
yarn create
yarn deploy
```

## Query a subgraph 
Once you subgraph is deployed, you can query it! To do so, you can directly write your query locally on http://localhost:8000/subgraphs/name/subraphtest/ (do not forget to update the url if you deployed your subgraph using another name than `subgraphtest`) or directly on `Subgraph Studio`.

Here is an example of query (based on our example) :

```graphQL
{
  usdtexchangevolumes(first:5) {
    id,
    amount
  }
}
```


## Publish a Subgraph
Now that you have deployed your subgraph, you can choose to publish it to the Graph Network!

```bash
yarn publish
```

## Local Graph Node Management (Optional)

### Reset node 
To reset and re-launch your local graph node, you can use the following script: 
```bash
yarn reset-graph-node
``` 

### Check Logs 
To check your graph-node logs, you can see the graph-node logs in the window running docker-compose, or you can fetch them from docker... 
