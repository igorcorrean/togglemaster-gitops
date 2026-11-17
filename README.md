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
do Datadog. Antes de aplicar a base, crie o Secret com a API key da conta
Datadog:

```bash
kubectl -n togglemaster create secret generic datadog-secret \
  --from-literal=api-key="$DD_API_KEY"
```

O Collector usa `datadoghq.com` por padrão. Para outra região, ajuste `DD_SITE`
no Deployment `otel-collector` em `apps/base/monitoring/otel-collector.yaml`.
O Alloy coleta logs dos Pods do namespace `togglemaster` e os envia ao Loki.

Para abrir o Grafana localmente:

```bash
kubectl -n togglemaster port-forward svc/grafana-service 3000:3000
```

Acesse `http://localhost:3000` e entre com `admin` / `admin`; o Grafana pedirá
a troca da senha no primeiro acesso. Prometheus fica disponível internamente
em `http://prometheus-service:9090`. Os dados do Prometheus usam armazenamento
efêmero e são retidos por até sete dias. No Grafana Explore, selecione Loki
como datasource e use, por exemplo, `{service="evaluation-service"}` para
consultar logs. Loki também usa armazenamento efêmero e retenção de sete dias.
Os traces ficam disponíveis no APM / Service Map do Datadog após configurar
a API key.