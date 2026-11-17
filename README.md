# togglemaster-gitops

## Observabilidade

O Kustomize da base instala Prometheus e Grafana no namespace `togglemaster`.
O Prometheus descobre os Services anotados com `prometheus.io/scrape: "true"`
e coleta `/metrics` a cada 15 segundos. Os cinco serviços expõem contagem e
duração de requisições HTTP; o `analytics-service` também expõe os resultados
do processamento de mensagens SQS.

Para abrir o Grafana localmente:

```bash
kubectl -n togglemaster port-forward svc/grafana-service 3000:3000
```

Acesse `http://localhost:3000` e entre com `admin` / `admin`; o Grafana pedirá
a troca da senha no primeiro acesso. Prometheus fica disponível internamente
em `http://prometheus-service:9090`. Os dados do Prometheus usam armazenamento
efêmero e são retidos por até sete dias.