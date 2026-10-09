# Demo: Prometheus + Grafana

Kit para demonstrar monitoramento e observabilidade de servidores em um seminário.

## Arquitetura

```
 Servidor (Linux)          coleta a cada 15s         consulta (PromQL)
┌───────────────┐   ┌──────────────────────┐   ┌──────────────────┐
│ Node Exporter │◄──│      Prometheus      │◄──│     Grafana      │
│   :9100       │   │ :9090 (TSDB+alertas) │   │ :3000 (dashboard)│
└───────────────┘   └──────────────────────┘   └──────────────────┘
```

- **Node Exporter** expõe métricas do host (CPU, memória, disco, rede).
- **Prometheus** busca (_pull_) essas métricas, guarda como séries temporais e avalia alertas.
- **Grafana** consulta o Prometheus e desenha os painéis.

O "emparelhamento" é feito automaticamente por _provisioning_:
`grafana/provisioning/datasources/prometheus.yml` aponta o Grafana para `http://prometheus:9090`
(o nome do serviço no Docker Compose), e o dashboard já vem carregado.

## Requisitos

- Docker e Docker Compose (Linux recomendado; no Windows/Mac o Node Exporter mede a VM do Docker).

## Subir

```bash
docker compose up -d
docker compose ps        # os 3 serviços devem estar "running"
```

| Serviço       | URL                           | Login             |
| ------------- | ----------------------------- | ----------------- |
| Grafana       | http://localhost:3000         | admin / seminario |
| Prometheus    | http://localhost:9090         | —                 |
| Node Exporter | http://localhost:9100/metrics | —                 |

Dashboard: Grafana → Dashboards → pasta **Seminário** → **Monitoramento do Servidor**.

## Encerrar

```bash
docker compose down        # para tudo
docker compose down -v     # para e apaga os dados
```

## Estrutura

```
demo-monitoramento/
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml       # alvos de coleta
│   └── alertas.yml          # regras de alerta
└── grafana/provisioning/
    ├── datasources/prometheus.yml   # liga Grafana → Prometheus
    └── dashboards/
        ├── dashboards.yml
        └── json/servidor.json       # dashboard da demo
```
