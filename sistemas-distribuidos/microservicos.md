# Arquiteturas de Microserviços

## 📖 Visão Geral

Microserviços são uma arquitetura que estrutura uma aplicação como uma coleção de serviços pequenos, autônomos e fracamente acoplados. Cada serviço é focado em uma capacidade específica de negócio.

## 🏛️ Princípios Fundamentais

### 1. Single Responsibility

Cada microserviço deve:
- Ter uma única responsabilidade bem definida
- Ser independentemente deployável
- Manter seu próprio banco de dados (database per service)

### 2. Loose Coupling

**Comunicação**:
- APIs REST/gRPC para síncrono
- Message queues (Kafka, RabbitMQ) para assíncrono
- Event-driven architecture

**Benefícios**:
- Mudanças localizadas
- Deploys independentes
- Escala seletiva

### 3. Autonomia

Cada serviço:
- Gerencia seus próprios dados
- Pode usar tecnologias diferentes (polyglot)
- Tem seu próprio ciclo de vida

## 🔒 Segurança em Microserviços

### Principle of Least Privilege (PoLP)

**Conceito**: Cada serviço deve ter apenas as permissões mínimas necessárias

**Implementação**:

```yaml
# Exemplo: Service Account com permissões limitadas
apiVersion: v1
kind: ServiceAccount
metadata:
  name: order-service-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: order-service-role
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
  # Apenas leitura de configs, sem write/delete
```

**Benefícios**:
- Reduz superfície de ataque
- Limita danos em caso de comprometimento
- Facilita auditoria

### Service Mesh para Segurança

**Istio/Linkerd**:
- mTLS automático entre serviços
- Políticas de autorização granulares
- Rate limiting e circuit breaking

```yaml
# Política de autorização Istio
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: payment-policy
spec:
  selector:
    matchLabels:
      app: payment-service
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/order-service-sa"]
    to:
    - operation:
        methods: ["POST"]
        paths: ["/api/v1/process-payment"]
```

## 🚀 Escalabilidade

### Horizontal Scaling

**Estratégia**: Adicionar mais instâncias do serviço

**Quando usar**:
- Serviços stateless
- Carga distribuída uniformemente
- Picos de tráfego previsíveis

**Kubernetes HPA**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### Vertical Scaling

**Estratégia**: Aumentar recursos (CPU, memória) de instâncias existentes

**Quando usar**:
- Serviços stateful (databases)
- Bottleneck em recursos específicos
- Limitações de concorrência

### Database Scaling Patterns

**Read Replicas**:
```
Write: Master DB
Reads: Distribuídas entre replicas
  → Replica 1 (região A)
  → Replica 2 (região B)
  → Replica 3 (região C)
```

**Sharding**:
```
Users 0-1000:   Shard 1
Users 1001-2000: Shard 2
Users 2001-3000: Shard 3
```

## 🌐 Platform Sustentável

### Microservice-Based Sustainable Platform

**Objetivos**:
- Eficiência energética
- Escala elástica (scale-to-zero quando idle)
- Otimização de recursos

**Implementação**:
```
Monitoring Service
  → Analisa métricas de uso
  → Identifica serviços ociosos
  → Escala para zero
  → Cold start quando necessário
```

**Tecnologias**:
- **Knative**: Serverless Kubernetes
- **KEDA**: Event-driven autoscaling
- **Green Software Foundation**: Métricas de carbono

## 🛠️ Patterns e Best Practices

### 1. API Gateway

**Responsabilidades**:
- Roteamento de requisições
- Autenticação/Autorização
- Rate limiting
- Request/response transformation

**Exemplo - Kong/Nginx**:
```nginx
location /api/orders {
  proxy_pass http://order-service:8080;
  limit_req zone=api_limit burst=10;
  proxy_set_header X-Request-ID $request_id;
}
```

### 2. Circuit Breaker

**Objetivo**: Prevenir cascading failures

**Estados**:
1. **Closed**: Requisições passam normalmente
2. **Open**: Falhas detectadas, requisições rejeitadas imediatamente
3. **Half-Open**: Testa se serviço recuperou

```python
# Exemplo com resilience4j
@CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
public PaymentResponse processPayment(PaymentRequest req) {
    return paymentClient.process(req);
}

public PaymentResponse paymentFallback(PaymentRequest req, Exception e) {
    // Retorna resposta degradada
    return PaymentResponse.queued();
}
```

### 3. Saga Pattern

**Problema**: Transações distribuídas em múltiplos serviços

**Solução**: Saga coordena sequência de transações locais

**Choreography**:
```
Order Created → [Event Bus]
  → Payment Service: Reserve Payment
  → Inventory Service: Reserve Stock
  → Shipping Service: Schedule Delivery

# Se falha:
Compensating transactions executadas em ordem reversa
```

**Orchestration**:
```
Saga Orchestrator:
  1. Chama Payment Service
  2. Se OK, chama Inventory Service
  3. Se OK, chama Shipping Service
  4. Se qualquer falha, executa compensações
```

### 4. Service Discovery

**DNS-based** (Kubernetes):
```
order-service.default.svc.cluster.local
  → Resolve para IPs dos pods
```

**Client-side** (Consul, Eureka):
```java
@LoadBalanced
@Bean
RestTemplate restTemplate() {
    return new RestTemplate();
}

// Chama usando service name
restTemplate.getForObject(
    "http://payment-service/api/process", 
    PaymentResponse.class
);
```

## 📊 Observabilidade

### Distributed Tracing

**Problema**: Rastrear requisição através de múltiplos serviços

**Solução - OpenTelemetry**:
```
Trace ID: 7a3d8f2b1c9e4a6d

Span 1: API Gateway [100ms]
  Span 2: Order Service [50ms]
    Span 3: Payment Service [30ms]
    Span 4: Inventory Service [20ms]
  Span 5: Notification Service [15ms]
```

**Proactive SLO Management**:
- Define SLOs por serviço (ex: p99 latency < 200ms)
- Alertas preditivos baseados em traces
- Identifica bottlenecks antes de afetar usuários

### Métricas Essenciais

**Golden Signals**:
1. **Latency**: Tempo de resposta
2. **Traffic**: Requisições por segundo
3. **Errors**: Taxa de erro
4. **Saturation**: Utilização de recursos

**Prometheus + Grafana**:
```promql
# Request rate
sum(rate(http_requests_total[5m])) by (service)

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) /
sum(rate(http_requests_total[5m]))

# P99 latency
histogram_quantile(0.99, 
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
```

## 🔧 Ferramentas e Frameworks

### Declaração de Linguagens Declarativas

**Problema**: Configuração complexa de arquiteturas distribuídas

**Solução**: Linguagens declarativas (Terraform, Pulumi)

```hcl
# Terraform - Infrastructure as Code
resource "kubernetes_deployment" "order_service" {
  metadata {
    name = "order-service"
  }
  spec {
    replicas = 3
    selector {
      match_labels = {
        app = "order-service"
      }
    }
    template {
      metadata {
        labels = {
          app = "order-service"
        }
      }
      spec {
        container {
          image = "myregistry/order-service:v1.2.0"
          port {
            container_port = 8080
          }
        }
      }
    }
  }
}
```

### Impact de IA Generativa em Times Ágeis

**Benefícios**:
- **Code generation**: Acelera desenvolvimento de boilerplate
- **Testing**: Geração automática de testes unitários
- **Documentation**: Mantem docs sincronizados com código

**Produtividade**:
- Até 40% redução em tempo de desenvolvimento
- Menos bugs em tarefas repetitivas
- Foco em lógica de negócio complexa

## 📚 Referências

- [Scalability in Microservices: Systematic Review](https://github.com/ElioNeto/agregador)
- [The Principle of Least Privilege in Microservices](https://github.com/ElioNeto/agregador)
- [Microservice-Based Platform for Sustainable Systems](https://github.com/ElioNeto/agregador)
- [From distributed tracing to proactive SLO management](https://github.com/ElioNeto/agregador)
- [Impacts of Generative AI on Agile Teams Productivity](https://github.com/ElioNeto/agregador)
- [Application of Declarative Languages in Distributed Architectures](https://github.com/ElioNeto/agregador)

## 🔗 Relacionado

- [Cloud e Edge Computing](cloud-edge.md)
- [Escalabilidade e Performance](escalabilidade.md)
- [Segurança em IoT](../seguranca/iot-security.md)
