# Indexer persistent data

- `mongo/`: existing MongoDB data for Exchange, Event Provider, and Explorer. Preserve this directory when changing deployment files.
- `dumps/exchangeV2`: optional Exchange gzip MongoDB archive.
- `dumps/eventProviderV2` or `dumps/eventProviderV2Pruned`: optional Event Provider archive; provide only one.
- `dumps/explorerV2`: optional Explorer archive.

Archives are restored only on the first MongoDB startup and removed after restoration. Genesis import uses the existing host path `~/.injectived/config/genesis.json`.
