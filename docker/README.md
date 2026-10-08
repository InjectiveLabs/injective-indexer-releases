# Injective Indexer

Docker setup for Exchange, Event Provider, and optional Explorer. MongoDB and MongoDB Exporter are included. Use Docker Compose v2 on a Linux host with journald.

Run all commands from this `docker` directory.

## Existing installations

Keep your existing `.env`, image versions, genesis files, and `volumes/mongo` data. Service names, database names, ports, the `injective` network name, and the `addons/docker-compose.dex.yaml` path are preserved. Missing optional variables have defaults, so the previous environment examples remain usable.

Apply the updated files in the existing deployment directory and keep the same Compose project name (`-p` or `COMPOSE_PROJECT_NAME`, if set). If moving to a new checkout, explicitly reuse the original MongoDB data directory; a new empty `volumes/mongo` starts a new database.

The node is now managed separately. An existing `injective-core` container can remain running on the `injective` network, with the same endpoints already configured in `.env`. Otherwise, set the existing `INDEXER_COSMOS_GRPC_ENDPOINTS`, `INDEXER_TENDERMINT_RPC_ENDPOINTS`, and `INDEXER_COSMOS_LCD_URL` variables to a reachable node, and update legacy `INDEXER_COSMOS_GRPC` / `INDEXER_TENDERMINT_RPC` if used. Match `INDEXER_COSMOS_TLS_ENABLED` to the connection.

**Omit `--remove-orphans` from old commands:** services removed from this configuration, including the existing node, must not be deleted during the transition. Do not set `COMPOSE_REMOVE_ORPHANS=true`. Node provisioning and node startup commands from the former guide no longer apply here.

Do not rerun genesis import or database initialization on an existing populated database. No database migration is introduced by this repository cleanup.

## Configure a new installation

```sh
# NETWORK is mainnet or testnet
cp .env.$NETWORK.example .env
```

Edit `.env` with your node connections, image versions, asset service addresses, and resource limits. The examples retain `injective-core` as the node hostname for existing installations; new installations must provide a reachable node. Set `INDEXER_STATSD_DISABLED=true` if no StatsD agent is available.

## MongoDB

```sh
docker compose -f docker-compose.yaml -f docker-compose.prod.yaml up -d mongo mongo-exporter
docker logs mongo -f
```

Wait for replica-set initialization to complete. On the first MongoDB startup, the initialization script restores gzip archives supplied in `volumes/dumps`: `exchangeV2`, `explorerV2`, and either `eventProviderV2` or `eventProviderV2Pruned`. Restored archives are removed. Existing database directories do not rerun initialization.

For genesis import into an empty Exchange database, retain the previous genesis path, `~/.injectived/config/genesis.json`, then run:

```sh
docker compose -f docker-compose.yaml run --rm indexer-exchange-import
```

Skip this import when restoring an initialized Exchange snapshot.

## Event Provider

The service definitions remain in the DEX addon:

```sh
docker compose -f docker-compose.yaml -f docker-compose.prod.yaml -f addons/docker-compose.dex.yaml up -d indexer-eventprovider-process indexer-eventprovider-api
docker logs -f indexer-eventprovider-process
```

Wait for initial sync to complete before starting consumers. Exchange and Explorer use `indexer-eventprovider-api:9910` by default; Exchange also retains `indexer-eventprovider-api:4444` for HTTP. If using an external Event Provider, keep your existing endpoint settings and skip starting these services.

## Exchange

```sh
docker compose -f docker-compose.yaml -f docker-compose.prod.yaml -f addons/docker-compose.dex.yaml up -d indexer-exchange-process indexer-exchange-api
docker logs -f indexer-exchange-process
```

Exchange API exposes HTTP on port `4444` and gRPC on `9910`.

## Optional Explorer

Start Explorer only when needed, using the same addon and service names as before:

```sh
docker compose -f docker-compose.yaml -f docker-compose.prod.yaml -f addons/docker-compose.dex.yaml up -d indexer-explorer-process indexer-explorer-api
```

No profile flag is required. Explorer uses `explorerV2` and exposes HTTP on port `4446` and gRPC on `9911`. Omitting the service names from `up` enables all services in the selected files, including Explorer and the one-time Exchange Import; use the explicit commands above for normal operation.

## Manage the indexer

```sh
docker compose -f docker-compose.yaml -f docker-compose.prod.yaml -f addons/docker-compose.dex.yaml restart
docker compose -f docker-compose.yaml -f docker-compose.prod.yaml -f addons/docker-compose.dex.yaml stop
docker compose -f docker-compose.yaml -f docker-compose.prod.yaml -f addons/docker-compose.dex.yaml down
```

Database files remain in `volumes/mongo`. A node still attached to the shared network may prevent Compose from removing that network; leave it attached while the node needs it.

## Makefile compatibility

The previous indexer commands remain available and read `.env.$NETWORK.example`, as before:

```sh
make init-$NETWORK
make eventprovider-$NETWORK
make relayer-$NETWORK
make reload-$NETWORK
make destroy-$NETWORK
```

`event-provider-$NETWORK`, as spelled in the previous guide, is also supported. `init` starts MongoDB and its exporter; `relayer` and `reload` start Exchange. Event Provider and Explorer are selected separately. Use `make explorer-$NETWORK` to enable Explorer.

Commands without a network suffix read `.env`: `make init`, `make eventprovider`, `make start`, `make explorer`, `make stop-explorer`, `make stop`, and `make destroy`. Override any configuration path with `ENV_FILE=/path/to/config.env`.

Existing `make follow-exchange-api`, `make follow-exchange-process`, and `make follow-eventprovider-process` commands are preserved. Explorer logs are available with `make follow-explorer-api` and `make follow-explorer-process`.
