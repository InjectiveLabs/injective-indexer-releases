# Indexer Changelog

## [Unreleased]

### Mainnet

- Pin the mainnet Docker examples for Exchange, Event Provider, and Explorer to indexer `v1.21.31` for Injective chain 1.21, including the corresponding `APP_VERSION` values.

For existing mainnet deployments, reconcile spot and derivative market state from the chain during this upgrade.

1. Set `INDEXER_COSMOS_LCD_URL` in your deployment `.env` to the base URL of a mainnet archival LCD reachable from Exchange Process. Before enabling the reload, verify that the LCD serves the last indexed block and the spot and derivative market queries at that same height using the `x-cosmos-block-height` header. The reload clears the two market collections in MongoDB before repopulating them.
2. Set `INDEXER_RELOAD_MARKETS_FROM_CHAIN=true` and recreate `indexer-exchange-process` with `v1.21.31` using that environment file. Run a single reload per Exchange database. Wait for `Markets migration completed successfully` in the process logs and confirm that indexing resumes.
3. Set `INDEXER_RELOAD_MARKETS_FROM_CHAIN=false` and recreate Exchange Process so later starts skip the rebuild. Complete the consumer upgrades before upgrading Event Provider API.

### Updated

- Pin the testnet Docker examples for Exchange, Event Provider, and Explorer to indexer `v1.21.31` for Injective chain 1.21, including the corresponding `APP_VERSION` values. This version preserves the `demolished` status of derivative markets when a settlement pause event arrives afterward.

### Upgrading an existing deployment

1. Copy the updated `EXCHANGE_*_VERSION`, `EVENTPROVIDER_*_VERSION`, and `EXPLORER_*_VERSION` values into your existing `.env`. Keep your node connections, genesis files, and MongoDB data. Make targets with a network suffix use the corresponding example file by default. Pass `ENV_FILE=.env` to use your existing configuration with those targets.
2. Recreate Exchange Process, Explorer Process if enabled, and any other consumers of your Event Provider with the new indexer version. Confirm that all consumer instances are ready before upgrading Event Provider API.
3. If you host Event Provider, recreate Event Provider Process and then Event Provider API. If you use a shared Event Provider, coordinate the API upgrade with its operator. Update Exchange API and Explorer API if enabled. Skip Exchange Import on an existing populated database.

### Deprecated

- Web3GW, Charts, and Chronos are deprecated in this repository and will no longer receive updates here. Their service definitions and dedicated environment variables have been removed. This repository maintains Exchange, Event Provider, and Explorer, with MongoDB and MongoDB Exporter.

### Migration

- Existing Web3GW, Charts, and Chronos containers can continue running, but this repository no longer manages them. Preserve their previous Compose configuration and environment files if you need to continue operating them.
- Omit `--remove-orphans` and do not enable `COMPOSE_REMOVE_ORPHANS` when updating an existing deployment: Compose can otherwise remove containers from the same project whose services are no longer defined, including the separately managed node.
