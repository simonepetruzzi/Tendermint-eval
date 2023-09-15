# Tendermint-eval
## Installation
| Requirements |      Note       | 
|-------------:|-----------------|
|  Go version  |Go 1.18 or higher|

First of all we need Tendermint installed, for this purpose let's see [installation instructions](https://github.com/tendermint/tendermint/blob/main/docs/introduction/install.md), alternatively you can run Tendermint in a Docker container by pulling the official image.

## Running Tendermint
Once Tendermint is installed run the following commands in different terminals:
```bash
tendermint node --home /path/to/your/testenet/node0 --proxy_app=kvstore
tendermint node --home /path/to/your/testenet/node1 --proxy_app=kvstore
tendermint node --home /path/to/your/testenet/node2 --proxy_app=kvstore
tendermint node --home /path/to/your/testenet/node3 --proxy_app=kvstore  
```
at this point you should see the four validators communicating each others and building blocks.

In order to inject transactions in the network, we use [tm-load-test tool](https://github.com/informalsystems/tm-load-test) by putting the following command:
```bash
./tm-load-test -c 1 -T 900 -r 100 -s 250 --broadcast-tx-method async --endpoints ws://tm-endpoint1.localhost:26660/websocket,ws://tm-endpoint2.localhost:26660/websocket
```
this will send transactions to the selected node(in this case the one exposing RPC on port 26660.

In order to collect metrics from Tendermint, we have to expose metrics. By default, these metrics are disabled and served on port 26660. If you want Tendermint exposing them you should enable it in the config file of each node and specify a different port (in my case 26666), since on 26660 is running RPC of the node1.

Once Tendermint metrics are exposed, we can run [Prometheus](https://github.com/prometheus/prometheus) in a Docker container by runnung the following command:
```bash
docker run --network host -p 9090:9090 prom/prometheus
```
**NOTE:** the --network host parameter is specified such that the container is built on the same network namespace of localhost.

At this point we should add a job in file prometheus.yml for scraping metrics from Tendermint:
```yml
- job_name: "tendermint"
    static_configs:
      - targets: ["host.docker.internal:26666"]
```
**NOTE:** host.docker.internal is used for dynamically resolve the localhost address (if you put localhost:26666 it will not work) 
