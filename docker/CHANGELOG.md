# Indexer Changelog

## [Unreleased]

### Updated

- Pin the testnet Docker examples for Exchange, Event Provider, and Explorer to indexer `v1.21.30` for Injective chain 1.21, including the corresponding `APP_VERSION` values.
- Pin the mainnet Docker examples for Exchange, Event Provider, and Explorer to indexer `v1.21.30` for Injective chain 1.21, including the corresponding `APP_VERSION` values.

### Upgrading an existing deployment to v1.21.30

1. Copy the updated `EXCHANGE_*_VERSION`, `EVENTPROVIDER_*_VERSION`, and `EXPLORER_*_VERSION` values into your existing `.env`. Keep your node connections, genesis files, and MongoDB data. Make targets with a network suffix use the corresponding example file by default. Pass `ENV_FILE=.env` to use your existing configuration with those targets.
2. Recreate Exchange Process, Explorer Process if enabled, and any other consumers of your Event Provider with the new indexer version. Confirm that all consumer instances are ready before upgrading Event Provider API.
3. If you host Event Provider, recreate Event Provider Process and then Event Provider API. If you use a shared Event Provider, coordinate the API upgrade with its operator. Update Exchange API and Explorer API if enabled. Skip Exchange Import on an existing populated database.

### Deprecated

- Web3GW, Charts, and Chronos are deprecated in this repository and will no longer receive updates here. Their service definitions and dedicated environment variables have been removed. This repository maintains Exchange, Event Provider, and Explorer, with MongoDB and MongoDB Exporter.

### Migration

- Existing Web3GW, Charts, and Chronos containers can continue running, but this repository no longer manages them. Preserve their previous Compose configuration and environment files if you need to continue operating them.
- Omit `--remove-orphans` and do not enable `COMPOSE_REMOVE_ORPHANS` when updating an existing deployment: Compose can otherwise remove containers from the same project whose services are no longer defined, including the separately managed node.
