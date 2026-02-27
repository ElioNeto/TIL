# Otimizações de Compactação em Bancos de Dados

## 📖 Introdução

A compactação (compaction) é um processo crítico em bancos de dados baseados em LSM Trees, responsável por mesclar, reorganizar e limpar dados armazenados em disco.

## 🎯 Objetivos da Compactação

1. **Liberar espaço em disco**
   - Remover registros deletados (tombstones)
   - Eliminar versões antigas de dados atualizados

2. **Melhorar performance de leitura**
   - Reduzir número de arquivos a consultar
   - Organizar dados para acesso sequencial

3. **Manter eficiência de escrita**
   - Equilibrar uso de recursos
   - Evitar acumulação excessiva de arquivos

## 🛠️ Estratégias de Compactação

### 1. Leveled Compaction

**Características**:
- Níveis organizados sem sobreposição de ranges
- Cada nível é ~10x maior que o anterior
- Compactação incremental

**Vantagens**:
- Menor space amplification
- Leituras mais rápidas (menos arquivos)
- Predictável para tuning

**Desvantagens**:
- Maior write amplification
- Mais I/O de compactação

**Usado em**: RocksDB, LevelDB

### 2. Tiered Compaction (STCS)

**Características**:
- Arquivos do mesmo tier podem ter ranges sobrepostos
- Merge acontece quando tier atinge threshold
- Menos compactações intermediárias

**Vantagens**:
- Menor write amplification
- Melhor throughput de escrita
- Menos I/O total

**Desvantagens**:
- Maior space amplification
- Leituras podem consultar mais arquivos

**Usado em**: Apache Cassandra, ScyllaDB

### 3. Time Window Compaction (TWCS)

**Foco**: Dados time-series

**Estratégia**:
- Agrupa dados por janelas de tempo
- Janelas antigas não são mais compactadas
- Facilita expiração de dados antigos (TTL)

**Ideal para**:
- Logs e eventos
- Métricas e monitoring
- Dados com TTL definido

## 🚀 Técnicas Avançadas

### Axis-Aware Layouts

**Conceito**: Organiza dados considerando padrões de acesso

**Implementação**:
```
1. Identifica "eixos" de acesso (temporal, espacial, etc.)
2. Layout de dados alinhado com esses eixos
3. Compactação otimizada para traversal patterns
```

**Benefícios**:
- Menos cache misses
- Melhor localidade de dados
- I/O mais eficiente

### Spherical Grid Storage (SGS)

**Aplicação**: Dados geoespaciais e espacio-temporais

**Inovação**:
- Coordenadas geográficas mapeadas para grid esférico
- Compactação respeita proximidade geográfica
- Queries espaciais muito mais rápidas

**Casos de uso**:
- GIS (Geographic Information Systems)
- Location-based services
- Climate data

### TrusterEfficient - Query-Oriented Clustering

**Objetivo**: Organizar dados baseado em padrões de query

**Abordagem**:
1. Analisa histórico de queries
2. Clusteriza dados frequentemente acessados juntos
3. Compactação preserva clustering

**Resultados**:
- Até 60% redução em query latency
- Menor I/O por query
- Cache hit rate maior

## ⚡ Otimizações de Performance

### 1. Parallel Compaction

**Estratégia**: Múltiplas compactações simultâneas

**Considerações**:
- Número de threads baseado em número de cores
- Evitar contenção de I/O
- Priorização de compactações críticas

**RocksDB Config**:
```cpp
options.max_background_compactions = 4;
options.max_subcompactions = 2;
```

### 2. Smart Scheduling

**Objetivo**: Executar compactação em momentos de baixa carga

**Técnicas**:
- **Throughput-aware**: Pausa compactação em picos de tráfego
- **Size-based**: Prioriza arquivos pequenos (rápidos)
- **Age-based**: Prioriza dados antigos (mais obsolescencia)

### 3. Bloom Filter Optimization

**Papel**: Evita leituras desnecessárias durante compactação

**Otimizações**:
- Bloom filters por SSTable
- False positive rate configurado por nível
- Cache de bloom filters em memória

**Configuração RocksDB**:
```cpp
BlockBasedTableOptions table_options;
table_options.filter_policy.reset(
    NewBloomFilterPolicy(10, false)); // 10 bits per key
```

## 📊 Monitoring e Tuning

### Métricas Importantes

1. **Write Amplification Factor (WAF)**
   ```
   WAF = Bytes escritos no disco / Bytes escritos pela aplicação
   ```
   - Ideal: 2-5x
   - Alto (>10x): Considerar tiered compaction

2. **Pending Compaction Bytes**
   - Acumulação indica bottleneck
   - Pode necessitar mais threads ou hardware

3. **Compaction Time per Level**
   - Identifica níveis problemáticos
   - Ajuda a otimizar size ratios

### Tuning Guidelines

**Para Write-Heavy Workloads**:
```
- Aumentar write buffer size
- Usar tiered compaction
- Reduzir número de níveis
- Level size multiplier maior (ex: 20)
```

**Para Read-Heavy Workloads**:
```
- Leveled compaction
- Mais memória para block cache
- Bloom filters agressivos
- Compactação mais frequente
```

**Para Balanced Workloads**:
```
- Hybrid approach (universal compaction)
- Dynamic level size adjustment
- Monitor e ajuste baseado em métricas
```

## 🔍 Debugging de Problemas

### Sintoma: Alto WAF

**Causas possíveis**:
- Size ratio muito pequeno entre níveis
- Compactação muito frequente
- Muitos updates/deletes

**Soluções**:
- Aumentar level size multiplier
- Considerar tiered compaction
- Implementar KV separation para large values

### Sintoma: Slow Queries

**Causas possíveis**:
- Muitos arquivos no L0
- Bloom filters insuficientes
- Compactação atrasada

**Soluções**:
- Aumentar L0 slowdown/stop triggers
- Mais memória para bloom filters
- Mais threads de compactação

### Sintoma: High Space Usage

**Causas possíveis**:
- Space amplification alta
- Muitos dados obsoletos
- Tiered compaction sem cleanup

**Soluções**:
- Mudar para leveled compaction
- Compactação manual forçada
- Implementar TTL para dados antigos

## 📚 Referências

- [Overlapping Aware Data Placement Optimizations for LSM Tree](https://github.com/ElioNeto/agregador)
- [Axis-Aware Layouts: Domain-Specific Traversal Strategies](https://github.com/ElioNeto/agregador)
- [Spherical Grid Storage (SGS): A Universal Spatio-Temporal System](https://github.com/ElioNeto/agregador)
- [TrusterEfficient: Query-oriented Clustered Storage Solution](https://github.com/ElioNeto/agregador)

## 🔗 Relacionado

- [LSM Trees](lsm-trees.md) - Arquitetura e conceitos fundamentais
- [MemTable](memtable.md) - Estrutura que alimenta a compactação
- [Log e Durabilidade](log&durabilidade.md) - Persistência antes da compactação
