ENV_FILE ?= .env
COMPOSE = docker compose --env-file $(ENV_FILE) -f docker-compose.yaml
DEX = $(COMPOSE) -f addons/docker-compose.dex.yaml

.PHONY: init import start reload eventprovider explorer stop-explorer stop destroy

init:
	$(COMPOSE) up -d mongo mongo-exporter

import:
	$(COMPOSE) run --rm indexer-exchange-import

start reload:
	$(COMPOSE) up -d mongo mongo-exporter indexer-exchange-process indexer-exchange-api

eventprovider:
	$(DEX) up -d mongo mongo-exporter indexer-eventprovider-process indexer-eventprovider-api

explorer:
	$(DEX) up -d indexer-explorer-process indexer-explorer-api

stop-explorer:
	$(DEX) stop indexer-explorer-process indexer-explorer-api

stop:
	$(DEX) stop

destroy:
	$(DEX) down

.PHONY: init-mainnet
init-mainnet: ENV_FILE = .env.mainnet.example
init-mainnet: init

.PHONY: eventprovider-mainnet
eventprovider-mainnet: ENV_FILE = .env.mainnet.example
eventprovider-mainnet: eventprovider

.PHONY: event-provider-mainnet
event-provider-mainnet: ENV_FILE = .env.mainnet.example
event-provider-mainnet: eventprovider

.PHONY: relayer-mainnet
relayer-mainnet: ENV_FILE = .env.mainnet.example
relayer-mainnet: start

.PHONY: reload-mainnet
reload-mainnet: ENV_FILE = .env.mainnet.example
reload-mainnet: reload

.PHONY: destroy-mainnet
destroy-mainnet: ENV_FILE = .env.mainnet.example
destroy-mainnet: destroy

.PHONY: explorer-mainnet
explorer-mainnet: ENV_FILE = .env.mainnet.example
explorer-mainnet: explorer

.PHONY: init-testnet
init-testnet: ENV_FILE = .env.testnet.example
init-testnet: init

.PHONY: eventprovider-testnet
eventprovider-testnet: ENV_FILE = .env.testnet.example
eventprovider-testnet: eventprovider

.PHONY: event-provider-testnet
event-provider-testnet: ENV_FILE = .env.testnet.example
event-provider-testnet: eventprovider

.PHONY: relayer-testnet
relayer-testnet: ENV_FILE = .env.testnet.example
relayer-testnet: start

.PHONY: reload-testnet
reload-testnet: ENV_FILE = .env.testnet.example
reload-testnet: reload

.PHONY: destroy-testnet
destroy-testnet: ENV_FILE = .env.testnet.example
destroy-testnet: destroy

.PHONY: explorer-testnet
explorer-testnet: ENV_FILE = .env.testnet.example
explorer-testnet: explorer

.PHONY: follow-mongo
follow-mongo:
	docker logs mongo -f

.PHONY: follow-mongo-exporter
follow-mongo-exporter:
	docker logs mongo-exporter -f

.PHONY: follow-exchange-import
follow-exchange-import:
	docker logs indexer-exchange-import -f

.PHONY: follow-exchange-process
follow-exchange-process:
	docker logs indexer-exchange-process -f

.PHONY: follow-exchange-api
follow-exchange-api:
	docker logs indexer-exchange-api -f

.PHONY: follow-eventprovider-process
follow-eventprovider-process:
	docker logs indexer-eventprovider-process -f

.PHONY: follow-eventprovider-api
follow-eventprovider-api:
	docker logs indexer-eventprovider-api -f

.PHONY: follow-explorer-process
follow-explorer-process:
	docker logs indexer-explorer-process -f

.PHONY: follow-explorer-api
follow-explorer-api:
	docker logs indexer-explorer-api -f
