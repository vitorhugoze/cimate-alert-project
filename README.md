# Climate Alert Project

Sistema de monitoramento climático com envio de alertas via WhatsApp, orquestrado via Kubernetes.

---

## Arquitetura

```
Google Climate API → climate-alert-producer → Broker Kafka → climate-alert-consumer → Waha (WhatsApp)
```

| Serviço | Tecnologia | Função |
|---|---|---|
| [`climate-alert-producer`](https://github.com/vitorhugoze/climate-alert-producer) | Java/Kafka | Consome dados climáticos e publica no Kafka |
| [`climate-alert-consumer`](https://github.com/vitorhugoze/climate-alert-consumer) | Java/Kafka | Lê mensagens do Kafka e aciona o Waha |
| `kafka` | Apache Kafka 3.8 | Broker do Kafka |
| `waha` | Waha `gows-2026.6.1` | API do WhatsApp |

---

## Deploy em Kubernetes

O diretório `kubernetes/` contém os manifestos na seguinte ordem de aplicação:

```bash
kubectl apply -f kubernetes/00-namespace.yaml      # Namespace: climate-alert
kubectl apply -f kubernetes/02-kafka.yaml          # Kafka (PVC 10Gi + Service)
kubectl apply -f kubernetes/03-waha.yaml           # Waha (PVCs 2Gi/5Gi + NodePort 32052)
kubectl apply -f kubernetes/04-consumer.yaml       # Consumer (2 réplicas + PVC 1Gi)
kubectl apply -f kubernetes/05-producer.yaml       # Producer (2 réplicas)
```

> **Antes do deploy**, crie os secrets conforme instruções em `kubernetes/01-criar-secrets.yaml`:
> - `climate-alert-secrets` — `WAHA_API_KEY`, `WAHA_PASSWORD`, `GOOGLE_CLIMATE_API_KEY`

Verifique o status:

```bash
kubectl get pods -n climate-alert
```

Acesse o dashboard do Waha através de: http://<node_internal_ip>:32052/dashboard/

Após isso crie uma sessão com o nome default, é através dela que as mensagens de WhatsApp serão enviadas

---

## Estrutura do Repositório

```
.
├── docker-compose.yaml
├── .gitignore
└── kubernetes/
    ├── 00-namespace.yaml
    ├── 01-criar-secrets.yaml   # Guia de criação de secrets (não aplicar diretamente)
    ├── 02-kafka.yaml
    ├── 03-waha.yaml
    ├── 04-consumer.yaml
    └── 05-producer.yaml
```

---

## Repositórios

- **climate-alert-producer** — https://github.com/vitorhugoze/climate-alert-producer
- **climate-alert-consumer** — https://github.com/vitorhugoze/climate-alert-consumer

---

## Imagens Utilizadas

- `devlikeapro/waha:gows-2026.6.1`
- `ghcr.io/vitorhugoze/climate-alert-consumer:latest`
- `ghcr.io/vitorhugoze/climate-alert-producer:latest`
- `apache/kafka:3.8.0`