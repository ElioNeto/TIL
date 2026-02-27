# Inteligência Artificial e Deep Learning

## 📖 Visão Geral

Deep Learning revolucionou a IA moderna, permitindo que modelos aprendam representações hierárquicas complexas de dados. Este documento compila aprendizados sobre arquiteturas, técnicas e aplicações de deep learning.

## 🧠 Arquiteturas Fundamentais

### Redes Neurais Convolucionais (CNNs)

**Aplicações**: Computer vision, image classification, object detection

**Conceitos-chave**:
- **Camadas convolucionais**: Extraem features locais
- **Pooling**: Reduz dimensionalidade preservando informação
- **Feature maps**: Representações hierárquicas da imagem

**Exemplo - Classificação Autônoma de Imagens**:
```
Arquitetura:
Input (224x224x3) 
  → Conv2D (64 filters) 
  → MaxPool 
  → Conv2D (128 filters)
  → MaxPool
  → Dense (1024)
  → Dropout (0.5)
  → Output (num_classes)
```

**Aplicações práticas**:
- Classificação de imagens médicas (raios-X, ressonância)
- Reconhecimento facial
- Inspeção de qualidade em manufatura

### Transformers e Attention Mechanisms

**Revolução**: Substituíram RNNs/LSTMs em muitas tarefas

**Self-Attention**:
- Cada token "atende" a todos os outros
- Captura dependências de longo alcance
- Paralelizável (ao contrário de RNNs)

**Multi-Head Attention**:
```python
# Conceptual
Q, K, V = query, key, value projections
Attention(Q,K,V) = softmax(QK^T / sqrt(d_k)) * V

# Multiple heads capture different patterns
MultiHead = Concat(head_1, ..., head_h) * W^O
```

**Aplicações**:
- NLP (BERT, GPT, T5)
- Computer vision (Vision Transformers - ViT)
- Time series forecasting

### Hybrid Architectures

**Spiking Neural Networks (SNNs) + Quantum**:
- Inspiradas no cérebro humano
- Eficiência energética superior
- Integração com computação quântica

**Parameter Efficient Hybrid**:
- Combina SNN com quantum data re-upload
- Reduz parâmetros tre using surrogate gradients
- Útil para hardware limitado (edge devices)

## 🚀 Técnicas de Otimização

### Continual Learning

**Desafio**: Aprender novas tarefas sem esquecer antigas (catastrophic forgetting)

**Shared LoRA Subspaces**:
- Low-Rank Adaptation (LoRA) para fine-tuning eficiente
- Subespaços compartilhados entre tarefas
- Almost Strict Continual Learning sem replay

**Benefícios**:
- Adaptação rápida a novos domínios
- Menor overhead de memória
- Preserva conhecimento anterior

### Combating Data and Target Shifts

**Problema**: Modelos degradam quando distribuição muda (distribution shift)

**Técnicas**:
1. **Domain Adaptation**
   - Alinha distribuições source e target
   - Adversarial training

2. **Test-Time Adaptation**
   - Ajusta modelo durante inferência
   - Sem acesso a labels

3. **Self-Training**
   - Pseudo-labels em dados target
   - Iterative refinement

### Enhancing Abstractiveness in Summarization

**Calibrated Distillation**:
- Treina modelo menor a partir de ensemble
- Calibração de confiança (temperature scaling)
- Gera sumários mais abstratos, menos extrativos

**Pipeline**:
```
1. Ensemble de modelos grandes gera distribuições
2. Calibração via temperature tuning
3. Distilação para modelo compacto
4. Fine-tuning com abstractiveness reward
```

## 🔬 Aplicações Específicas

### Computer Vision

#### Flow3r - Visual Geometry Learning

**Inovação**: Factored flow prediction escalável

**Arquitetura**:
- Prediz optical flow entre frames
- Decomposição fatorada para escala
- Aplicações em 3D reconstruction

**Casos de uso**:
- Robótica (navegação visual)
- Carros autônomos
- AR/VR tracking

#### VideoWorld 2 - Transferable Knowledge

**Objetivo**: Aprender representações de vídeos real-world transferíveis

**Abordagem**:
- Self-supervised learning em large-scale video
- Temporal consistency constraints
- Transfer para downstream tasks (action recognition, etc.)

### Ciências Biológicas

#### Chlamy_ChloroPred

**Aplicação**: Predição de proteínas de cloroplasto

**Método**:
- Deep learning binary classifier
- Alta acuracybelt para Chlamydomonas reinhardtii
- Potencial cross-proteome versatility

**Importância**:
- Acelera pesquisa em bioenergia
- Identificação de alvos farmacológicos
- Engenharia de microalgas

### Imagens Médicas

#### Water-Unsuppressed MRSI

**Desafio**: MRI spectroscopic imaging com alta resolução

**Solução DL**:
- Deep learning para reconstrução de imagens
- Ultra-high field (7T+)
- Quantificação simultânea de:
  - Metabólitos
  - Susceptibilidade magnética
  - Myelin water

**Benefícios clínicos**:
- Diagnóstico precoce de doenças neurológicas
- Monitoramento de tratamentos
- Pesquisa em neurociência

## ⚡ Reinforcement Learning

### AutoGrid AI

**Domínio**: Gerenciamento autônomo de microgrids

**Framework**:
- Deep Reinforcement Learning
- Multi-objective optimization:
  - Custo de energia
  - Emissões de carbono
  - Confiabilidade

**Algoritmo**:
```
Estado: [energy_demand, prices, battery_soc, solar_gen, grid_status]
Ações: [charge_battery, discharge_battery, buy_grid, sell_grid]
Reward: -cost - carbon_penalty + reliability_bonus
```

**Resultados**:
- Até 30% redução em custos vs baseline
- Menor pegada de carbono
- Maior resiliência

### Multi-Agent Systems

**Potential Games Analysis**:
- Framework teórico para coordenação
- Price of Anarchy bounds
- Conexão com contrastive learning

**CommCP - Communication with Conformal Prediction**:
- Agentes coordenam via comunicação baseada em LLM
- Conformal prediction para uncertainty quantification
- Eficiente mesmo com comunicação limitada

## 📊 Physics-Informed Neural Networks (PINNs)

### Conceito

Integra leis físicas (EDPs) diretamente na loss function:

```python
# Loss total
L_total = L_data + λ_physics * L_physics + λ_boundary * L_boundary

# Exemplo: Equação de calor
# ∂u/∂t = α * ∂²u/∂x²
L_physics = MSE(du_dt - alpha * d2u_dx2, 0)
```

### Implementação com Deep Learning

**Vantagens**:
- Menos dados necessários
- Soluções fisicamente plausíveis
- Generaliza melhor para novos regimes

**Aplicações**:
- Mecânica dos fluidos
- Simulações de materiais
- Previsão climática

## 🛠️ Ferramentas e Frameworks

### Desenvolvimento e Debugging

**PATe 2.0** - Histopathological Education:
- Plataforma escalável para treinamento
- Deep learning para assisted learning
- Smart annotation tools

### Dataset Development

**A Very Big Video Reasoning Suite**:
- Large-scale benchmark para video understanding
- Múltiplas tarefas (reasoning, temporal, spatial)
- Desafia SOT models

## 📚 Referências

### Arquiteturas e Técnicas
- [AI-Driven Autonomous Image Classification](https://github.com/ElioNeto/agregador)
- [Flow3r: Factored Flow Prediction](https://github.com/ElioNeto/agregador)
- [Shared LoRA Subspaces for Continual Learning](https://github.com/ElioNeto/agregador)
- [Parameter efficient hybrid spiking-quantum CNN](https://github.com/ElioNeto/agregador)

### Aplicações
- [AutoGrid AI: Deep RL for Microgrid Management](https://github.com/ElioNeto/agregador)
- [Chlamy_ChloroPred: Chloroplast Protein Prediction](https://github.com/ElioNeto/agregador)
- [Deep learning water-unsuppressed MRSI](https://github.com/ElioNeto/agregador)
- [VideoWorld 2: Learning from Real-world Videos](https://github.com/ElioNeto/agregador)

### Teoria
- [On the Analysis of Potential Games](https://github.com/ElioNeto/agregador)
- [Implementing physics-informed neural networks](https://github.com/ElioNeto/agregador)
- [Combating Data and Target Shifts in Visual Tasks](https://github.com/ElioNeto/agregador)

## 🔗 Relacionado

- [LLMs e Modelos de Linguagem](llms.md)
- [Computer Vision](computer-vision.md)
- [GPU e Aceleração](../hardware/gpu-aceleracao.md)
