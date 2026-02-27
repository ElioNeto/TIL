# Blockchain, Criptografia e Segurança

## 📖 Visão Geral

Blockchain e criptografia formam a base de sistemas seguros modernos, desde criptomoedas até supply chain e autenticação. Este documento compila conceitos e aplicações práticas.

## 🔒 Fundamentos de Blockchain

### Arquitetura Básica

**Componentes**:
1. **Blocos**: Contêm transações e metadata
2. **Hash Chain**: Cada bloco referencia o hash do anterior
3. **Consensus**: Mecanismo de acordo distribuído
4. **Smart Contracts**: Código executável na blockchain

```
Bloco N-1        Bloco N          Bloco N+1
[────────]  →   [────────]  →   [────────]
|Prev Hash|      |Prev Hash|      |Prev Hash|
|Txs      |      |Txs      |      |Txs      |
|Nonce    |      |Nonce    |      |Nonce    |
|Timestamp|      |Timestamp|      |Timestamp|
[────────]      [────────]      [────────]
```

### Propriedades de Segurança

**Imutabilidade**:
- Alterar bloco requer recalcular todos os blocos seguintes
- Computacionalmente inviável em redes grandes

**Descentralização**:
- Sem single point of failure
- Resistência a censura
- Transparência verificable

**Consensus Mechanisms**:
1. **Proof of Work (PoW)**: Bitcoin, Ethereum (antigo)
2. **Proof of Stake (PoS)**: Ethereum 2.0, Cardano
3. **Practical Byzantine Fault Tolerance (PBFT)**: Hyperledger

## 🔐 Criptografia Quântico-Resistente

### Ameaça Quântica

**Algoritmo de Shor**:
- Quebra RSA e ECC em tempo polinomial
- Computadores quânticos suficientemente grandes ameaçam criptografia atual

### QBUILD - Quantum-Resistant Blockchain

**Arquitetura**:
```
Camada de Aplicação
  ↓
Smart Contracts (Quantum-Safe)
  ↓
Consensus Layer
  │
  ├─ Post-Quantum Signatures (CRYSTALS-Dilithium)
  ├─ Lattice-based Encryption
  └─ Hash-based Merkle Trees
  ↓
P2P Network Layer
```

**Algoritmos Pós-Quânticos**:

1. **CRYSTALS-Dilithium** (assinaturas):
   - Baseado em lattices
   - Chave pública: ~1.3KB
   - Assinatura: ~2.4KB

2. **CRYSTALS-Kyber** (key exchange):
   - Encapsulação de chave
   - Resistência quântica comprovada

3. **SPHINCS+** (hash-based signatures):
   - Stateless (ao contrário de XMSS)
   - Segurança baseada apenas em funções hash

### Implementação

```python
# Exemplo conceitual com liboqs (Open Quantum Safe)
from oqs import Signature

# Gera par de chaves quantum-safe
sig = Signature("Dilithium3")
public_key = sig.generate_keypair()

# Assina mensagem
message = b"Transaction data"
signature = sig.sign(message)

# Verificação
is_valid = sig.verify(message, signature, public_key)
```

## 🏗️ Aplicações Empresariais

### Combating Foreign Bribery and Corruption

**Problema**: Corrupção em supply chains globais

**Solução Integrada**:
```
Blockchain (imutabilidade)
  +
Corporate Governance (políticas)
  +
IA (detecção de anomalias)
  +
Sustainability Metrics
```

**Benefícios**:
- Rastreabilidade completa de transações
- Auditoria automática via smart contracts
- Alerta de padrões suspeitos via IA

### Blockchain para Contabilidade Moderna

**Transformação**:
- **Tradicional**: Livros contábeis centralizados, reconciliação manual
- **Blockchain**: Ledger distribuído, reconciliação automática

**Triple Entry Accounting**:
```
Empresa A (débito)  →  Blockchain Entry  ←  Empresa B (crédito)
                         (immutable)
```

**Vantagens**:
- Elimina discrepâncias
- Auditoria em tempo real
- Redução de fraudes

## 🚗 Veículos Elétricos e IoT

### Vehicle-to-Vehicle (V2V) Power Trading

**Cenário**: VEs compartilham energia via blockchain

**Arquitetura**:
```
VE A (vendedor)
  ↓
Smart Contract
  │
  ├─ Verifica saldo de bateria
  ├─ Negocia preço
  ├─ Executa transferência
  └─ Registra transação
  ↓
VE B (comprador)
```

**Trustworthiness**:
- Proof of battery state via IoT sensors
- Reputação on-chain dos participantes
- Dispute resolution via oracle

### Integração IA + Blockchain para Supply Chain

**Pipeline**:
```
1. Sensores IoT coletam dados
   ↓
2. IA valida qualidade/autenticidade
   ↓
3. Blockchain registra evento verificado
   ↓
4. Smart contract atualiza estado
```

**Casos de uso**:
- **Farmácia**: Rastreabilidade de medicamentos
- **Alimentos**: Cold chain monitoring
- **Luxo**: Autenticação de produtos

## 🔍 Detecção de Ameaças

### Enhancing Ransomware Detection

**Abordagem com IA**:

1. **Behavioral Analysis**:
   ```
   Features:
   - Taxa de criptografia de arquivos
   - Padrões de acesso a disco
   - Network traffic anomalies
   - Process creation patterns
   ```

2. **Machine Learning Models**:
   - Random Forest para classificação
   - LSTM para detecção temporal
   - Ensemble para maior acurácia

3. **Response Automation**:
   ```python
   if ransomware_detected:
       isolate_machine()  # Quarentena de rede
       snapshot_memory()  # Forense
       rollback_files()   # Recuperação via backups
       alert_soc()        # Notifica equipe
   ```

**Métricas**:
- **Precision**: >95% (poucos falsos positivos)
- **Recall**: >98% (detecta maioria dos ataques)
- **Detection Time**: <5 segundos

### Windows-APT 2025 Dataset

**Objetivo**: Dataset para treinar/testar detecção de APTs

**Conteúdo**:
- Attack scenarios inspirados em APTs reais
- Logs de sistema (Event Logs, Sysmon)
- Network captures (PCAPs)
- Malware samples

**Scenarios Incluidos**:
1. **Initial Access**: Spear phishing
2. **Execution**: PowerShell, WMI abuse
3. **Persistence**: Registry keys, scheduled tasks
4. **Privilege Escalation**: Token manipulation
5. **Defense Evasion**: Obfuscation, process injection
6. **Lateral Movement**: Pass-the-Hash, RDP
7. **Exfiltration**: DNS tunneling, HTTPS covert channel

### Luminous Defense - XAI para IoT

**Desafio**: Infraestruturas críticas IoT precisam de defesa explicavel

**XAI (Explainable AI)**:
```
Modelo de Detecção
  ↓
Predição: "Ataque detectado"
  +
Explicação: "Traffic spike de 1000% no sensor X,
              padrão similar ao ataque conhecido Y"
```

**Benefícios**:
- Analistas entendem razão do alerta
- Reduz falsos positivos investigados
- Facilita compliance e auditoria

**Técnicas XAI**:
- **SHAP**: Feature importance
- **LIME**: Local interpretability
- **Attention Maps**: Para modelos de deep learning

## 🌐 Segurança em Edge Computing

### Desafios Específicos

**Edge vs Cloud**:
- Recursos limitados (CPU, memória)
- Conexão intermitente
- Acesso físico mais fácil
- Ambientes hostis

### Security and Privacy in Edge - Electric Vehicles

**Ameaças**:
1. **Spoofing**: Falsificação de identidade do veículo
2. **Tampering**: Modificação de dados de sensor
3. **Information Disclosure**: Vazamento de localização/hábitos
4. **DoS**: Ataques aos sistemas de controle

**Mitigações**:

```
1. Secure Boot + TPM
   - Garante integridade de firmware

2. Hardware Security Module (HSM)
   - Chaves criptográficas protegidas

3. Encrypted Communication
   - TLS 1.3 para V2X
   - End-to-end encryption

4. Differential Privacy
   - Adiciona ruído aos dados de localização
   - Preserva utilidade agregada
```

### Towards Smarter IoT

**Taxonomia de Segurança IoT**:

```
Physical Layer
  ├─ Tamper resistance
  └─ Side-channel protection

Network Layer
  ├─ Secure protocols (CoAP-DTLS)
  └─ Intrusion detection

Application Layer
  ├─ Access control
  └─ Data encryption

Management
  ├─ Secure updates (OTA)
  └─ Key management
```

**Direções Futuras**:
- AI-powered anomaly detection no edge
- Blockchain para device identity
- Federated learning para privacy

## 🛡️ Cyber Stress Management

### Problema

Profissionais de cibersegurança sofrem alto stress:
- Alertas 24/7
- Pressão de resposta rápida
- Impacto de falhas elevado

### Soluções Organizacionais

**Técnicas**:
1. **Automação**: SOAR platforms reduzem tarefas repetitivas
2. **Rotação**: Evita fadiga emão on-call
3. **Treinamento**: Simulações regulares reduzem estresse real
4. **Resiliência Psicológica**: Programas de bem-estar

**Métricas de Sucesso**:
- Redução em burnout rate
- Melhor retenção de talentos
- Maior qualidade de resposta a incidentes

## 📚 Referências

### Blockchain e Criptografia
- [QBUILD: Quantum-Resistant Blockchain Architecture](https://github.com/ElioNeto/agregador)
- [Blockchain and Cyber security in Modern Accounting](https://github.com/ElioNeto/agregador)
- [Trustworthy V2V Power Trading Scheme](https://github.com/ElioNeto/agregador)
- [Combating Foreign Bribery via Blockchain + AI](https://github.com/ElioNeto/agregador)
- [Integrating AI and Blockchain for Supply Chain](https://github.com/ElioNeto/agregador)

### Detecção de Ameaças
- [Enhancing Ransomware Detection Using AI](https://github.com/ElioNeto/agregador)
- [Windows-APT 2025 Dataset](https://github.com/ElioNeto/agregador)
- [Luminous Defense: XAI for Critical IoT](https://github.com/ElioNeto/agregador)

### IoT e Edge Security
- [Security and Privacy in Edge Computing - Electric Vehicles](https://github.com/ElioNeto/agregador)
- [Towards Smarter IoT: Taxonomy and Prospective Directions](https://github.com/ElioNeto/agregador)

### Gestão
- [Cyber Stress Management Among Employees](https://github.com/ElioNeto/agregador)
- [Cybersecurity Disclosure and Digital Maturity](https://github.com/ElioNeto/agregador)

## 🔗 Relacionado

- [Segurança em IoT](iot-security.md)
- [Microserviços - Segurança](../sistemas-distribuidos/microservicos.md#segurança)
- [Computação Quântica](../hardware/quantum.md)
