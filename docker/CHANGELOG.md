# Indexer Changelog

## [Unreleased]

### Deprecated

- Web3GW, Charts, and Chronos are deprecated in this repository and will no longer receive updates here. Their service definitions and dedicated environment variables have been removed. This repository maintains Exchange, Event Provider, and Explorer, with MongoDB and MongoDB Exporter.

### Migration

- Existing Web3GW, Charts, and Chronos containers can continue running, but this repository no longer manages them. Preserve their previous Compose configuration and environment files if you need to continue operating them.
- Omit `--remove-orphans` and do not enable `COMPOSE_REMOVE_ORPHANS` when updating an existing deployment: Compose can otherwise remove containers from the same project whose services are no longer defined, including the separately managed node.
