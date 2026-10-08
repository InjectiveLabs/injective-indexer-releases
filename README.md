# Injective Indexer Releases

Docker configuration for Exchange, Event Provider, and Explorer, with MongoDB and MongoDB Exporter.

- `docker/docker-compose.yaml`: MongoDB, MongoDB Exporter, Exchange Import, Exchange Process, and Exchange API.
- `docker/addons/docker-compose.dex.yaml`: Event Provider Process/API and Explorer Process/API.

The DEX addon retains the paths and service names used by the previous setup guide. Explorer is optional: start only the services you need. No Compose profiles are required.

See the [setup and migration guide](docker/README.md) and [changelog](docker/CHANGELOG.md).
