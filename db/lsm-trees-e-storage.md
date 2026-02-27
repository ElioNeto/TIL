# LSM Trees e Sistemas de Armazenamento

## Visão Geral

LSM (Log-Structured Merge) Trees são estruturas de dados fundamentais para sistemas de armazenamento modernos, especialmente bancos de dados NoSQL e key-value stores como RocksDB, LevelDB e Cassandra.

## Conceitos Principais

### Estrutura Básica

- **Memtable**: Buffer em memória que recebe escritas
- **SSTables**: Arquivos imutáveis em disco organizados em níveis
- **Write-Ahead Log (WAL)**: Garante durabilidade das operações
- **Bloom Filters**: Aceleram buscas reduzindo I/O desnecessário

### Processo de Compactação

A compactação é o processo de mesclar SSTables para:
- Remover dados deletados e versões antigas
- Reduzir amplificação de leitura
- Organizar dados de forma mais eficiente

**Estratégias de Compactação:**
- **Leveling**: Mantém níveis organizados com razão de tamanho
- **Tiering**: Agrupa SSTables por idade
- **Híbrido**: Combina abordagens para otimizar diferentes workloads

## Otimizações Avançadas

### C2LSM - Configuration Paradigm

Sistema que adapta configurações de compactação dinamicamente baseado no workload, reduzindo write amplification.

### LSM HB-Tree

Combina LSM Tree com B-Tree hierárquica para melhorar desempenho de consultas por intervalo (range queries).

### RGKV - GPU-Empowered Compaction

Utiliza GPUs para acelerar o processo de compactação, aproveitando paralelismo massivo para operações de merge e ordenação.

### Overlapping-Aware Placement

Otimiza a colocação de dados considerando sobreposição de chaves entre SSTables, reduzindo I/O durante compactação.

### Enhancing Merge Efficiency

Técnicas de coordenação de runs ordenados para melhorar eficiência do merge:
- Pré-processamento de metadados
- Paralelização de operações de merge
- Cache inteligente de blocos

## Armazenamento Especializado

### Tidehunter - Large-Value Storage

Solução otimizada para valores grandes, minimizando relocação de dados durante compactação.

### Truster - Query-Oriented Storage

Armazenamento clusterizado otimizado para consultas específicas, organizando dados baseado em padrões de acesso.

### GPComp - SSD-GPU Peer-to-Peer

Utiliza DMA direto entre SSD e GPU para acelerar compactação, eliminando overhead da CPU.

## Sistemas Geoespaciais

### Spherical Grid Storage (SGS)

Sistema de coordenadas universais espaço-temporais para dados geoespaciais em escala planetária.

## Trade-offs Principais

### Write Amplification
- Dados escritos múltiplas vezes durante compactação
- Impacto: Desgaste de SSD, latência de escrita
- Mitigação: Otimização de políticas de compactação

### Read Amplification
- Múltiplas SSTables precisam ser lidas
- Impacto: Latência de leitura
- Mitigação: Bloom filters, índices, cache

### Space Amplification
- Dados obsoletos ocupam espaço até compactação
- Impacto: Uso de disco
- Mitigação: Compactação mais agressiva

## Boas Práticas

1. **Monitore métricas de amplificação** constantemente
2. **Ajuste políticas de compactação** para seu workload específico
3. **Use Bloom filters** adequadamente dimensionados
4. **Configure memória suficiente** para memtables e cache
5. **Considere hardware especializado** (GPU, SmartNIC) para workloads intensivos

## Tendências Futuras

- **Compactação orientada por IA**: Previsão de padrões de acesso
- **Hardware especializado**: Aceleradores dedicados para compactação
- **Armazenamento hierárquico**: Integração com NVMe, Optane e SCM
- **Compactação adaptativa**: Ajuste dinâmico baseado em workload

## Referências

Baseado em pesquisas do repositório [agregador](https://github.com/ElioNeto/agregador):
- C2LSM: A Configuration Paradigm for Efficient Compaction
- LSM HB-Tree: Hierarchical B-Tree Augmentation
- RGKV: GPU-Empowered Compaction Framework
- Rethinking LSM-tree based Key-Value Stores: A Survey
- E diversos outros papers sobre otimizações de LSM Trees

---
*Última atualização: 2026-02-27*