# Testes, Debugging e Desenvolvimento de Software

## 📖 Visão Geral

Práticas modernas de desenvolvimento exigem ferramentas e técnicas sofisticadas para garantir qualidade, debugar problemas e manter produtividade.

## 🧪 Geração Automática de Testes

### Rapid End-to-End Test Generation

**Desafio**: Testes E2E são demorados para criar manualmente

**Solução com LLMs**:
```
User Story
  ↓
LLM (GPT-4, Claude)
  ├─ Gera test scenarios
  ├─ Cria test code (Playwright, Selenium)
  └─ Adiciona assertions
  ↓
Executa testes
  ↓
Feedback Loop
  ├─ Falhas → Refina prompts
  └─ Sucesso → Adiciona a suite
```

**Exemplo Prompt**:
```
Crie um teste E2E para:
- Login com credenciais válidas
- Navegar para dashboard
- Verificar presença de widget "Sales"
- Fazer logout

Framework: Playwright + TypeScript
```

**Output**:
```typescript
import { test, expect } from '@playwright/test';

test('user can login and view dashboard', async ({ page }) => {
  await page.goto('https://app.example.com/login');
  
  await page.fill('[name="email"]', 'user@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL(/.*dashboard/);
  await expect(page.locator('.widget-sales')).toBeVisible();
  
  await page.click('[data-testid="logout-button"]');
  await expect(page).toHaveURL(/.*login/);
});
```

### Hallucination Mitigation

**Problema**: LLMs podem gerar testes com seletores incorretos

**Soluções**:

1. **DOM Snapshot Validation**:
   ```
   LLM gera teste → Valida seletores contra DOM real
   ```

2. **Iterative Refinement**:
   ```python
   for attempt in range(max_retries):
       test_code = llm.generate(prompt)
       result = run_test(test_code)
       if result.passed:
           break
       prompt += f"\nErro: {result.error}. Corrija o teste."
   ```

3. **Self-Healing Tests**:
   - Testes detectam elementos por múltiplos seletores
   - Fallback automático se um falha

## 🔍 Debugging Avançado

### Automated Multi-Source Debugging

**Fontes de Informação**:
```
Error Message
  +
Stack Trace
  +
Logs (aplicativos, sistema)
  +
Métricas (CPU, memória, rede)
  +
Code Context
  ↓
LLM Analysis
  ↓
Root Cause + Fix Suggestion
```

**Natural Language Error Explanation**:

**Input**:
```
Error: NullPointerException
  at UserService.getProfile(UserService.java:42)
  at ProfileController.show(ProfileController.java:18)

Code (linha 42):
return user.getAddress().getCity();
```

**Output**:
```
Problema: Null pointer ao acessar cidade do endereço

Causa Raiz:
- Campo 'address' de 'user' é null
- Provavelmente usuário sem endereço cadastrado

Correção Sugerida:
if (user.getAddress() != null) {
    return user.getAddress().getCity();
}
return "Cidade não cadastrada";

Prevenção:
- Usar Optional<Address> no modelo
- Validar na criação de usuário
- Adicionar constraint de banco de dados
```

### Root Cause Analysis com LLMs

**Método**:

1. **Correlation Analysis**:
   ```
   Timeline:
   12:00 - Deploy da versão 2.1.0
   12:05 - Spike em erro 500
   12:10 - Aumento de 3x em latency de DB
   
   → LLM identifica: Deploy introduziu N+1 query
   ```

2. **Code Diff Analysis**:
   ```diff
   - for (User user : users) {
   -     user.getOrders();  // 1 query por user
   - }
   + users = userRepo.findAllWithOrders();  // 1 query total
   ```

3. **Knowledge Base Search**:
   - Busca incidentes similares passados
   - Sugere soluções que funcionaram

## 💡 AnoMod - Anomaly Detection Dataset

### Objetivo

Dataset para treinar modelos de detecção de anomalias em sistemas

### Conteúdo

**Cenrios Incluidos**:
1. **Memory Leak**
   - Métricas: Memória crescente linearmente
   - Sintomas: OOM errors eventual

2. **CPU Thrashing**
   - Métricas: Context switches elevados
   - Sintomas: Latency alta, throughput baixo

3. **Network Saturation**
   - Métricas: Packet loss, retransmissions
   - Sintomas: Timeouts, failures intermitentes

4. **Database Deadlocks**
   - Métricas: Lock wait time
   - Sintomas: Transações travadas

### Uso para Treinamento

```python
import pandas as pd
from sklearn.ensemble import IsolationForest

# Carrega dataset
df = pd.read_csv('anomod/memory_leak.csv')

# Features
X = df[['mem_used', 'mem_growth_rate', 'gc_frequency']]

# Treina detector
model = IsolationForest(contamination=0.1)
model.fit(X)

# Predição
y_pred = model.predict(X_test)
# -1 = anomalia, 1 = normal
```

## 🛠️ Ferramentas Modernas

### AgentCgroup - Controle de Recursos de AI

**Problema**: Agentes de IA consomem recursos descontroladamente

**Solução**: Cgroups para limitar recursos

```bash
# Cria cgroup para agente IA
sudo cgcreate -g cpu,memory:ai-agent

# Limita CPU a 50% e memória a 2GB
sudo cgset -r cpu.cfs_quota_us=50000 ai-agent
sudo cgset -r memory.limit_in_bytes=2G ai-agent

# Executa agente no cgroup
sudo cgexec -g cpu,memory:ai-agent python ai_agent.py
```

**Benefícios**:
- Previne que agente monopolize recursos
- Coexistência com outras aplicações
- Melhor previsibilidade de performance

### Uncertainty Modeling (SysML v2)

**Aplicação**: Modelagem de sistemas com incerteza

**Exemplo - Sistema de Navegação**:
```sysml
part def NavigationSystem {
  // Parâmetros com incerteza
  attribute gps_accuracy : Real 
    { distribution = Normal(mean=5.0, std=2.0); }
  
  attribute sensor_noise : Real
    { distribution = Exponential(lambda=0.1); }
  
  // Propaga incerteza
  calc position_uncertainty {
    return sqrt(gps_accuracy^2 + sensor_noise^2);
  }
}
```

**Benefícios**:
- Especificação formal de incertezas
- Simulação Monte Carlo automática
- Validação de requisitos probabilisticos

## 📊 Impacto de IA Generativa

### Produtividade em Times Ágeis

**Estudo Multi-Caso**:

**Ganhos Medidos**:
1. **Code Generation**: 35-45% mais rápido
   - Boilerplate (controllers, models)
   - Testes unitários
   - Documentação

2. **Code Review**: 25% mais rápido
   - IA sugere melhorias
   - Identifica bugs potenciais
   - Verifica best practices

3. **Debugging**: 30% mais rápido
   - Explicações de erros
   - Sugestões de fix
   - Contexto de stack trace

**Desafios**:
- **Over-reliance**: Desenvolvedores podem aceitar sugestões sem entender
- **Quality Variance**: Sugestões nem sempre corretas
- **Security**: Risco de gerar código vulnerável

**Best Practices**:
```
1. Sempre revisar código gerado
2. Usar IA como assistente, não substituto
3. Validar sugestões com testes
4. Treinar time em limitações de LLMs
```

## 🔍 Penetration Testing com LLMs

### What Makes a Good LLM Agent for Pentest?

**Capacidades Necessárias**:

1. **Reconnaissance**
   ```
   - Port scanning (nmap)
   - Service enumeration
   - Vulnerability scanning (Nessus, OpenVAS)
   ```

2. **Exploitation**
   ```
   - Metasploit integration
   - Custom exploit development
   - Payload generation
   ```

3. **Post-Exploitation**
   ```
   - Privilege escalation
   - Lateral movement
   - Data exfiltration
   ```

**Exemplo - Agente Autônomo**:
```python
class PentestAgent:
    def __init__(self, target):
        self.target = target
        self.llm = ChatGPT4()
        self.tools = ["nmap", "metasploit", "sqlmap"]
    
    def recon(self):
        prompt = f"""
        Target: {self.target}
        Objetivo: Enumeração inicial
        Tools: {self.tools}
        
        Gere comandos de reconnaissance.
        """
        commands = self.llm.generate(prompt)
        results = [execute(cmd) for cmd in commands]
        return self.analyze_results(results)
    
    def exploit(self, vulnerabilities):
        for vuln in vulnerabilities:
            prompt = f"""
            Vulnerabilidade: {vuln}
            Gere exploit strategy.
            """
            strategy = self.llm.generate(prompt)
            if self.execute_exploit(strategy):
                return True
        return False
```

**Limitações**:
- LLMs podem sugerir ações ilégicas
- Requer validação humana constante
- Ética: Uso apenas em ambientes autorizados

## ⚡ Observabilidade

### From Distributed Tracing to Proactive SLO

**Evolução**:
```
Traditional Monitoring (reativo)
  ↓
Distributed Tracing (diagnóstico)
  ↓
SLO Management (proativo)
```

**Proactive SLO**:

1. **Define SLOs**:
   ```yaml
   slo:
     - metric: request_latency_p99
       target: 200ms
       window: 30d
     - metric: error_rate
       target: 0.1%
       window: 30d
   ```

2. **Burn Rate Analysis**:
   ```
   Error Budget Remaining: 70%
   Current Burn Rate: 5%/day
   
   → Alerta: At este ritmo, error budget esgota em 14 dias
   ```

3. **Predictive Alerts**:
   ```
   ML Model previu:
   - Latency aumentará 20% na próxima hora
   - SLO será violado se nenhuma ação for tomada
   
   Ações Sugeridas:
   - Escalar replicas do serviço X
   - Aumentar cache TTL
   ```

## 📚 Referências

### Testes e Debugging
- [Rapid End-to-End Test Generation and Hallucination Mitigation](https://github.com/ElioNeto/agregador)
- [Automated Multi-Source Debugging](https://github.com/ElioNeto/agregador)
- [Root Cause Analysis Method Based on LLMs](https://github.com/ElioNeto/agregador)
- [AnoMod: Dataset for Anomaly Detection](https://github.com/ElioNeto/agregador)

### Ferramentas
- [AgentCgroup: Controlling OS Resources of AI Agents](https://github.com/ElioNeto/agregador)
- [Uncertainty Modeling for SysML v2](https://github.com/ElioNeto/agregador)

### Produtividade
- [Impacts of Generative AI on Agile Teams Productivity](https://github.com/ElioNeto/agregador)
- [What Makes a Good LLM Agent for Real-world Penetration Testing](https://github.com/ElioNeto/agregador)

### Observabilidade
- [From distributed tracing to proactive SLO management](https://github.com/ElioNeto/agregador)

## 🔗 Relacionado

- [Microserviços - Observabilidade](../sistemas-distribuidos/microservicos.md#observabilidade)
- [IA Generativa](../ia-e-ml/ia-deep-learning.md)
- [Segurança - Penetration Testing](../seguranca/blockchain-cripto.md)
