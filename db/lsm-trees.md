# LSM Trees e Key-Value Stores

## 📖 Visão Geral

LSM (Log-Structured Merge) Trees são estruturas de dados fundamentais para bancos de dados modernos como RocksDB, LevelDB, Apache Cassandra e muitos outros sistemas de armazenamento chave-valor.

## 💡 Conceitos Principais

### Arquitetura Básica

As LSM Trees organizam dados em múltiplos níveis (levels):
- **Nível 0 (L0)**: Recebe dados diretamente da MemTable quando ela atinge o limite
- **Níveis seguintes (L1, L2, ...)**: Dados organizados e compactados progressivamente
- Cada nível é tipicamente 10x maior que o anterior

### Processo de Compactação (Compaction)

A compactação é o processo crítico que:
1. **Mescla arquivos SSTable** de diferentes níveis
2. **Remove dados obsoletos** (deletes, overwrites)
3. **Reordena chaves** para otimizar buscas
4. **Libera espaço** em disco

#### Estratégias de Compactação

**Leveled Compaction** (usado no RocksDB):
- Mantém níveis não sobrepostos
- Menor amplificação de espaço
- Melhor para workloads de leitura

**Tiered Compaction** (usado no Cassandra):
- Permite sobreposição entre arquivos do mesmo nível
- Menor amplificação de escrita
- Melhor para workloads de escrita intensiva

## 🚀 Otimizações Modernas

### 1. C2LSM - Configuration Paradigm

**Problema**: Configuração de compactação é complexa e impact dependencies workload

**Solução**: Paradigma de configuração adaptativo que:
- Ajusta parâmetros dinamicamente baseado no padrão de acesso
- Reduz write amplification
- Melhora throughput geral

### 2. LSM HB-Tree

**Inovação**: Aumenta LSM Tree com B-Tree hierárquica

**Benefícios**:
- Acelera operações de range queries
- Mantém eficiência de escrita da LSM
- Índice hierárquico para navegação rápida

### 3. RGKV - GPU-Empowered Compaction

**Abordagem**: Usa GPGPU para acelerar compactação

**Resultados**:
- Até 3x mais rápido que compactação tradicional em CPU
- Paraleliza ordenação e merge de SSTables
- Ideal para grandes volumes de dados

### 4. Overlapping-Aware Data Placement

**Objetivo**: Minimizar sobreposição de dados entre níveis

**Técnicas**:
- Posicionamento inteligente de SSTables
- Redução de I/O durante compactação
- Melhora latency de leitura

### 5. Tidehunter - Large-Value Storage

**Foco**: Otimização para valores grandes (>KB)

**Estratégia**:
- Separação de chave-valor (KV separation)
- Valores grandes armazenados separadamente
- Minimiza movimentação de dados durante compactação

## ⚡ Trade-offs Fundamentais

LSM Trees enfrentam três trade-offs principais:

### Write Amplification (WA)
- Quantidade de dados escritos vs dados originais
- Causado por múltiplas compactações
- **Mitigação**: Tiered compaction, write buffer maior

### Read Amplification (RA)
- Número de arquivos consultados para uma leitura
- Maior em níveis superiores
- **Mitigação**: Bloom filters, índices, caching

### Space Amplification (SA)
- Espaço usado vs dados reais
- Causado por dados obsoletos não compactados
- **Mitigação**: Leveled compaction, compactação agressiva

## 🛠️ Técnicas de Otimização de Merge

### Coordinated Sorted Run Merge

**Conceito**: Coordena merge entre SSTables para minimizar I/O

**Implementação**:
```
1. Identifica SSTables com alta sobreposição
2. Prioriza merge desses arquivos
3. Usa algoritmos de merge multi-way eficientes
4. Aproveita cache de páginas do sistema operacional
```

**Ganhos**: Até 40% de redução em tempo de compactação

### GPComp - GPU Peer-to-Peer DMA

**Inovação**: Transferência direta SSD → GPU sem passar pela CPU

**Pipeline**:
1. SSD transfere dados diretamente para memória GPU via PCIe
2. GPU processa compactação (ordenação, merge)
3. Resultados escritos de volta ao SSD

**Vantagens**:
- Elimina bottleneck de memória CPU
- Reduz latência de transferência
- Escala bem com múltiplas GPUs

## 📈 Aplicações Práticas

### Quando Usar LSM Trees

✅ **Ideal para**:
- Workloads write-heavy (logs, métricas, eventos)
- Time-series databases
- Sistemas de streaming
- Armazenamento de objetos

❌ **Menos adequado para**:
- Queries complexas com JOINs
- Atualizações in-place frequentes
- Leituras aleatórias de pequenos valores

### Bancos de Dados que Usam LSM

1. **RocksDB** (Meta)
   - Leveled compaction
   - Usado em MySQL, MongoDB, CockroachDB

2. **Apache Cassandra**
   - Tiered compaction (STCS)
   - Distribuído nativamente

3. **LevelDB** (Google)
   - Implementação original de referencia
   - Base para RocksDB

4. **ScyllaDB**
   - LSM em C++ com otimizações para hardware moderno
   - Shard-per-core architecture

## 📚 Referências

- [Rethinking LSM-tree based Key-Value Stores: A Survey](https://github.com/ElioNeto/agregador)
- [C2LSM: A Configuration Paradigm for Efficient Compaction](https://github.com/ElioNeto/agregador)
- [Enhancing LSM trees merge efficiency via coordinated sorted runs](https://github.com/ElioNeto/agregador)
- [RGKV: A GPGPU-Empowered Compaction Framework](https://github.com/ElioNeto/agregador)
- [Tidehunter: Large-Value Storage With Minimal Data Relocation](https://github.com/ElioNeto/agregador)

## 🔗 Relacionado

- [MemTable](memtable.md) - Estrutura em memória que alimenta a LSM Tree
- [Log e Durabilidade](log&durabilidade.md) - WAL para persistência
- [Compactação](compactacao.md) - Detalhes sobre estratégias de compaction
