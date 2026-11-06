# Indexer Changelog

## [Unreleased]

### Updated

- Pin Exchange (Import, Process, and API), Event Provider, and Explorer to [indexer v1.21.8](https://github.com/InjectiveLabs/injective-indexer/releases/tag/v1.21.8) for chain 1.21 in both network examples, including their `APP_VERSION` values.
- Existing installations must copy the `EXCHANGE_*_VERSION`, `EVENTPROVIDER_*_VERSION`, and `EXPLORER_*_VERSION` values into their own environment file to select this release. Preserve node connections, genesis files, and database data.

### Deprecated

- Web3GW, Charts, and Chronos are deprecated in this repository and will no longer receive updates here. Their service definitions and dedicated environment variables have been removed. This repository maintains Exchange, Event Provider, and Explorer, with MongoDB and MongoDB Exporter.

### Migration

- Existing Web3GW, Charts, and Chronos containers can continue running, but this repository no longer manages them. Preserve their previous Compose configuration and environment files if you need to continue operating them.
- Omit `--remove-orphans` and do not enable `COMPOSE_REMOVE_ORPHANS` when updating an existing deployment: Compose can otherwise remove containers from the same project whose services are no longer defined, including the separately managed node.
