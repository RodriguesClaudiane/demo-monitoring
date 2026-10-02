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
- **Prometheus** busca (*pull*) essas métricas, guarda como séries temporais e avalia alertas.
- **Grafana** consulta o Prometheus e desenha os painéis.

O "emparelhamento" é feito automaticamente por *provisioning*:
`grafana/provisioning/datasources/prometheus.yml` aponta o Grafana para `http://prometheus:9090`
(o nome do serviço no Docker Compose), e o dashboard já vem carregado.

## Requisitos

- Docker e Docker Compose (Linux recomendado; no Windows/Mac o Node Exporter mede a VM do Docker).

## Subir

```bash
docker compose up -d
docker compose ps        # os 3 serviços devem estar "running"
```

| Serviço       | URL                          | Login               |
|---------------|------------------------------|---------------------|
| Grafana       | http://localhost:3000        | admin / seminario   |
| Prometheus    | http://localhost:9090        | —                   |
| Node Exporter | http://localhost:9100/metrics| —                   |

Dashboard: Grafana → Dashboards → pasta **Seminário** → **Monitoramento do Servidor**.

## Roteiro sugerido para a apresentação (~15 min)

1. **Métricas cruas** – abra `http://localhost:9100/metrics` e mostre o formato texto
   (ex.: `node_cpu_seconds_total`). "É isso que um exporter entrega."
2. **Prometheus coletando** – em `http://localhost:9090/targets`, mostre os alvos `UP`.
3. **PromQL ao vivo** – na aba *Graph* do Prometheus, rode:
   ```promql
   node_memory_MemAvailable_bytes
   rate(node_network_receive_bytes_total[1m])
   100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)
   ```
4. **Grafana** – mostre o data source em *Connections → Data sources → Prometheus*
   (botão *Save & test*) e depois abra o dashboard.
5. **Gerar carga** e ver o painel reagir (atualiza a cada 5s):
   ```bash
   # CPU: estressa todos os núcleos por 60s
   docker run --rm -it alpine sh -c "apk add -q stress-ng && stress-ng --cpu 0 --timeout 60s"
   # Memória: aloca ~1 GB por 60s
   docker run --rm -it alpine sh -c "apk add -q stress-ng && stress-ng --vm 1 --vm-bytes 1G --timeout 60s"
   ```
6. **Alertas** – em `http://localhost:9090/alerts`, mostre `CPUAlta` passar de
   *inactive* → *pending* → *firing* durante o estresse.
7. **Queda do servidor** – `docker stop node-exporter`: o painel "Status" fica **FORA**
   e o alerta `ServidorFora` dispara. Depois `docker start node-exporter`.

## Dica: dashboard pronto da comunidade

Em *Dashboards → New → Import*, digite o ID **1860** (Node Exporter Full) e escolha
o data source Prometheus — é um ótimo contraste com o painel simples criado aqui.

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
