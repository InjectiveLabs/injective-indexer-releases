# Injective Indexer

Docker setup for Exchange, Event Provider, and optional Explorer. MongoDB and MongoDB Exporter are included. Use Docker Compose v2 on a Linux host with journald.

Run all commands from this `docker` directory.

The current unreleased configuration targets **Injective chain v1.21**. Chain versions and indexer image tags are independent: the examples currently pin indexer `v1.21.31`. Confirm the selected images support your chain release before deployment; do not derive an image tag from the chain version.

## Existing installations

Keep your existing `.env`, image versions, genesis files, and `volumes/mongo` data. Service names, database names, ports, the `injective` network name, and the `addons/docker-compose.dex.yaml` path are preserved. Missing optional variables have defaults, so the previous environment examples remain usable.

Apply the updated files in the existing deployment directory and keep the same Compose project name (`-p` or `COMPOSE_PROJECT_NAME`, if set). If moving to a new checkout, explicitly reuse the original MongoDB data directory; a new empty `volumes/mongo` starts a new database.

The node is now managed separately. An existing `injective-core` container can remain running on the `injective` network, with the same endpoints already configured in `.env`. Otherwise, set the existing `INDEXER_COSMOS_GRPC_ENDPOINTS`, `INDEXER_TENDERMINT_RPC_ENDPOINTS`, and `INDEXER_COSMOS_LCD_URL` variables to a reachable node, and update legacy `INDEXER_COSMOS_GRPC` / `INDEXER_TENDERMINT_RPC` if used. Match `INDEXER_COSMOS_TLS_ENABLED` to the connection.

**Omit `--remove-orphans` from old commands:** services removed from this configuration, including the existing node, must not be deleted during the transition. Do not set `COMPOSE_REMOVE_ORPHANS=true`. Node provisioning and node startup commands from the former guide no longer apply here.

Do not rerun genesis import or database initialization on an existing populated database. Follow the [version-specific upgrade steps](CHANGELOG.md), including the mainnet market reconciliation required for the current release.

If an upgrade adds Event Provider events, update every consuming process, including Exchange and any enabled Explorer, before updating Event Provider API. Wait until every consumer replica supports the new events. The startup order below applies to a new installation using mutually compatible images, not to rolling upgrades.

## Configure a new installation

```sh
NETWORK=testnet # or mainnet
cp .env.$NETWORK.example .env
```

Edit `.env` with your node connections, image versions, asset service addresses, and resource limits. The examples retain `injective-core` as the node hostname for existing installations; new installations must provide a reachable node. Set `INDEXER_STATSD_DISABLED=true` if no StatsD agent is available.

Exchange API loads denoms from the asset service during startup. If the configured asset gRPC endpoint fails, configure a working `INDEXER_ASSETS_SERVICE_URL` and set `ASSET_SERVICE_USE_HTTP=true` to use its HTTP API. Recreate the Exchange containers after changing `.env`; `docker compose restart` does not apply changed environment variables.

The example MongoDB limits are sized for servers: mainnet uses a 70 GiB memory limit and 56 GiB WiredTiger cache; testnet uses 32 GiB and 24 GiB respectively. Size these settings for the host and dataset, leaving memory for the indexer processes and APIs. Docker Desktop also needs a logging override because these Compose files use `journald`; the published indexer `v1.21.31` image requires `linux/amd64` (emulation on ARM hosts).

## Prepare snapshots before the first startup

For an existing network, obtain MongoDB backups for the chosen network and compatible indexer version before starting MongoDB. A chain-node snapshot is not a MongoDB indexer backup. Confirm each backup's indexed height, the Event Provider's retained block range, and available disk space for both the compressed archives and restored databases. Event Provider must retain the blocks needed after each consumer's restored height; it also needs a node that can serve blocks after its own restored height.

```sh
mkdir -p volumes/dumps
cp /path/to/exchange-backup.archive.gz volumes/dumps/exchangeV2
cp /path/to/eventprovider-backup.archive.gz volumes/dumps/eventProviderV2
# Only when enabling Explorer:
cp /path/to/explorer-backup.archive.gz volumes/dumps/explorerV2
```

Use gzip-compressed `mongodump --archive` files, not tarballs or database directories. For a backup whose database is named `eventProviderV2Pruned`, use `volumes/dumps/eventProviderV2Pruned` instead of `eventProviderV2`; the initializer renames its collections during restore. Supply only one Event Provider archive. Keep a separate copy of every backup: successful restores remove the staged archives.

This repository does not publish backups or a verified download URL. Obtain them from your snapshot provider. Starting without snapshots requires the matching genesis and a node retaining all necessary historical blocks; a pruned public node is not sufficient to replay missing history.

Archives must be present **before the first MongoDB startup**. Adding them after `volumes/mongo` has been initialized does not trigger a restore. For a fresh installation accidentally initialized without backups, stop the stack and move its data directory aside before retrying with the archives; never replace an existing installation's database this way.

## MongoDB

```sh
docker compose -f docker-compose.yaml -f docker-compose.prod.yaml up -d mongo mongo-exporter
docker logs mongo -f
```

Wait for replica-set initialization to complete. On the first MongoDB startup, the initialization script restores gzip archives supplied in `volumes/dumps`: `exchangeV2`, `explorerV2`, and either `eventProviderV2` or `eventProviderV2Pruned`. Restored archives are removed. Existing database directories do not rerun initialization.

Wait for `MongoDB init process complete` in the logs, then verify that MongoDB is writable and that the expected databases were restored:

```sh
docker exec mongo mongosh --quiet --eval 'db.hello().isWritablePrimary'
docker exec mongo mongosh --quiet --eval 'db.adminCommand({listDatabases: 1, nameOnly: true})'
```

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

Exchange API exposes HTTPS on port `4444` with the configured TLS certificate and gRPC on `9910`.

## Optional Explorer

Start Explorer only when needed, using the same addon and service names as before:

```sh
docker compose -f docker-compose.yaml -f docker-compose.prod.yaml -f addons/docker-compose.dex.yaml up -d indexer-explorer-process indexer-explorer-api
```

No profile flag is required. Explorer uses `explorerV2` and exposes HTTPS on port `4446` with the configured TLS certificate and gRPC on `9911`. Omitting the service names from `up` enables all services in the selected files, including Explorer and the one-time Exchange Import; use the explicit commands above for normal operation.

## Verify synchronization

```sh
docker compose -f docker-compose.yaml -f docker-compose.prod.yaml -f addons/docker-compose.dex.yaml ps -a
curl --fail http://localhost:9216/metrics
# Local diagnostics with the certificate bundled in the example image:
curl --fail --insecure https://localhost:4443/api/event_provider/v1/latestheight
curl --fail --insecure https://localhost:4444/api/health/v1/status
# Only when enabling Explorer:
curl --fail --insecure https://localhost:4446/api/health/v1/status
```

Use `--insecure` only for these localhost checks; deployed API clients should validate a trusted certificate. Confirm `mongodb_up` is `1`, Event Provider height advances, and each enabled consumer catches up. Inspect the health response body as well as its HTTP status: an API can accept requests while its database has no indexed data. Check process logs for missing or pruned blocks, connection failures, and repeated restarts before considering the stack ready.

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
