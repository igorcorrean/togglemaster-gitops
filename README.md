# togglemaster-gitops

## Observabilidade

O Kustomize da base instala Prometheus, Grafana, Loki, Alloy e OpenTelemetry
Collector no namespace `togglemaster`.
O Prometheus descobre os Services anotados com `prometheus.io/scrape: "true"`
e coleta `/metrics` a cada 15 segundos. Os cinco serviços expõem contagem e
duração de requisições HTTP; o `analytics-service` também expõe os resultados
do processamento de mensagens SQS.

Os serviços exportam spans OTLP ao Collector. O contexto W3C é propagado entre
os serviços HTTP e nas mensagens SQS; o Collector encaminha os traces ao APM
do Datadog e ao Tempo, disponível no Grafana. Antes de aplicar a base, crie o
Secret com a API key da conta Datadog:

```bash
kubectl -n togglemaster create secret generic datadog-secret \
  --from-literal=api-key="$DD_API_KEY"
```

O Collector usa `datadoghq.com` por padrão. Para outra região, ajuste `DD_SITE`
no Deployment `otel-collector` em `apps/base/monitoring/otel-collector.yaml`.
O Alloy coleta logs dos Pods do namespace `togglemaster` e os envia ao Loki.
No Grafana, Prometheus fornece métricas, Loki fornece logs e Tempo fornece
traces; as três fontes estão disponíveis no Explore.

Para abrir o Grafana localmente:

```bash
kubectl -n togglemaster port-forward svc/grafana-service 3000:3000
```

Acesse `http://localhost:3000` e entre com `admin` / `admin`; o Grafana pedirá
a troca da senha no primeiro acesso. No Explore, selecione Loki e use, por
exemplo, `{service="evaluation-service"}` para consultar logs; selecione Tempo
para consultar traces. Prometheus fica disponível internamente em
`http://prometheus-service:9090`. Prometheus e Loki usam armazenamento
efêmero; Loki e Tempo têm retenção de até sete dias. Os traces também ficam
disponíveis no APM / Service Map do Datadog após configurar a API key.