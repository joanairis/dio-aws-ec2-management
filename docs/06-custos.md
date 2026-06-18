# 💰 Otimização de Custos em EC2

Estratégias para reduzir gastos com instâncias EC2 mantendo performance.

---

## 💵 Modelos de Precificação

### 1. On-Demand

**Como funciona**: Pague por hora/segundo, sem compromisso

```yaml
t3.micro: ~$0.0104 por hora
t3.small: ~$0.0208 por hora
m6i.large: ~$0.096 por hora

Caso de uso: Testes, desenvolvimento, cargas impredizíveis
```

### 2. Reserved Instances (RI)

**Como funciona**: Compromisso de 1 ou 3 anos = desconto

```yaml
Termos:
  - 1 ano, All Upfront: 24% desconto
  - 1 ano, Partial Upfront: 20% desconto
  - 3 anos, All Upfront: 62% desconto ⭐
  - 3 anos, Partial Upfront: 57% desconto

Exemplo:
  t3.medium On-Demand: $0.0416/hora × 24h × 365d = $364/ano
  t3.medium 3yr RI: $0.0156/hora × 24h × 365d = $137/ano
  
  Economia: 62% ($227/ano)
```

**Quando usar**:
- ✅ Workloads estáveis (produção)
- ✅ Previsibilidade de longo prazo
- ✅ Aplicações sempre ligadas

---

### 3. Spot Instances

**Como funciona**: Capacidade ociosa com desconto de até 90%

```yaml
Preço típico Spot:
  t3.micro: $0.003/hora (71% desconto!)
  m6i.large: $0.035/hora (63% desconto)

⚠️ Desvantagem: Podem ser interrompidas com 2 minutos de aviso
```

**Quando usar**:
- ✅ Batch processing (análise de dados)
- ✅ Renderização
- ✅ Indexação
- ✅ Testes e CI/CD
- ❌ Aplicações em produção (geralmente)

**Mitigar Interrupção**:
```yaml
Usar Spot Fleets:
  - Instância 1: t3.micro Spot
  - Instância 2: t3.small Spot
  - Instância 3: t3.medium On-Demand (fallback)
  
Se uma cai, outra assume!
```

---

### 4. Savings Plans

**Como funciona**: Compromisso de gastos (não tipo específico)

```yaml
Exemplo:
  Plano: $0.50/hora × 365 dias × 3 anos = $1,314 total
  Cobre qualquer combinação de instâncias EC2
  Máximo desconto: 72%
```

**Flexibilidade**:
- ✅ Trocar tipo de instância dentro do plano
- ✅ Trocar região
- ✅ Trocar SO (Linux ↔ Windows)

---

## 🎯 Estratégia de Custo Híbrida

```
Produção:
  50% Reserved Instances (estáveis)
  30% On-Demand (crescimento)
  20% Spot Instances (batch jobs)

Desenvolvimento:
  100% Spot Instances ← Máximo desconto!

Resultado: Economia de 40-60%
```

---

## 📊 Free Tier AWS

```yaml
Vigência: 12 meses após criar conta

Instâncias EC2:
  t2.micro: 750 horas/mês ✅ GRÁTIS

Armazenamento EBS:
  30 GB gp3: GRÁTIS
  
Transferência de dados:
  15 GB/mês saída: GRÁTIS

Data de expiração: Importante!
→ Ativar billing alerts antes de perder free tier
```

---

## 🔍 Analisar e Reduzir Custos

### AWS Cost Explorer

```
1. AWS Console → Cost Explorer
2. Filtrar: EC2
3. Ver custos por:
   - Tipo de instância
   - Região
   - Período (diário/mensal)
4. Identificar outliers
```

### Right Sizing

```
Passo 1: Analisar uso
  - CPU: Average 10%, Peak 60%
  - Memória: Average 2GB, Peak 4GB
  
Passo 2: Comparar com tipo atual
  - Atual: m6i.xlarge ($0.192/h)
  - Necessário: t3.small ($0.0208/h)
  
Passo 3: Redimensionar
  - Economia: 89% ($1,355/ano)
```

### Compute Optimizer (Automático)

```
AWS Console → Compute Optimizer

→ Recomendações de tipo:
  Instância atual: m5.large (subutilizada)
  Recomendação: t3.medium
  Economia: 45% ($520/ano)
```

---

## 💡 Dicas de Economia

### 1. Encerrar Instâncias Não Utilizadas

```bash
# Identificar instâncias paradas por > 30 dias
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=stopped" \
  --query 'Reservations[*].Instances[*].[InstanceId,State.Name,LaunchTime]'

# Terminar (⚠️ sem recuperação!)
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0
```

### 2. Usar Auto Scaling

```yaml
Configuração:
  Mínimo: 1 instância (tráfego baixo)
  Máximo: 10 instâncias (pico)
  Desejado: Automático baseado em CPU

Resultado:
  - Noite: 1 instância (economia)
  - Dia: 3-5 instâncias (conforme demanda)
  - Pico: 10 instâncias (escalado automaticamente)
```

### 3. Consolidar Aplicações

```yaml
Antes (4 instâncias t3.medium):
  Aplicação Web: t3.medium
  Cache: t3.medium
  Batch Job: t3.medium
  Database: t3.medium
  Custo: $0.0832/h
  
Depois (2 instâncias m6i.large + Elasticache):
  Web+Cache: m6i.large
  Database gerenciado: RDS
  Custo: $0.050/h

Economia: 40%
```

### 4. Usar Spot Instances para Não-Críticos

```bash
# Template de launch para Spot
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --instance-market-options "MarketType=spot,SpotOptions={MaxPrice=0.005}"
```

### 5. Monitorar Elastic IPs Não Usados

```bash
# IPs elásticos não associados = custos!
# Desassociar ou liberar IPs não usados

aws ec2 describe-addresses \
  --filters "Name=association-state-name,Values=disassociated"

# Liberar
aws ec2 release-address --allocation-id eipalloc-xxxxx
```

### 6. Otimizar Armazenamento EBS

```yaml
Antes: 100 GB gp2
  Custo: 100 × $0.10 = $10/mês

Depois: 50 GB gp3 (suficiente)
  Custo: 50 × $0.08 = $4/mês
  
Economia: $6/mês ($72/ano) com 50% menos custo
```

### 7. Desativar Monitoramento Detalhado

```
Se não precisa de métricas a cada 1 minuto:

Console EC2 → Instância → Ações → Monitoramento
→ "Desativar monitoramento detalhado"

Economia: $0.10 por métrica/mês
```

---

## 📊 Calculadora de Preço

### Exemplo Customizado

```yaml
Cenário: Servidor web em produção

Instância:
  Tipo: m6i.large
  Região: us-east-1
  Preço On-Demand: $0.096/hora
  Preço 3yr RI: $0.036/hora

Armazenamento:
  100 GB gp3: $8/mês

Data Transfer:
  50 GB/mês saída: $4.50 (primeiros 1GB grátis)

Total:
  On-Demand: $0.096 × 24 × 30 = $69.12 + $8 + $4.50 = $81.62/mês
  Reserved: $0.036 × 24 × 30 = $25.92 + $8 + $4.50 = $38.42/mês
  Economia com RI: 53% ($518/ano)
```

### Usar AWS Pricing Calculator

```
1. AWS Console → Pricing Calculator
2. Adicione serviços:
   - EC2 instances
   - EBS volumes
   - Data transfer
3. Compare On-Demand vs RI vs Spot
```

---

## 🚨 Alertas de Custos

### Configurar Budget Alerts

```
AWS Console → Budgets

1. "Criar budget"
2. Nome: EC2-Monthly-Budget
3. Limite: $100/mês
4. Alerta: > $80 (80% do limite)
5. Notificação: Email
```

### Cost Anomaly Detection

```
AWS Console → Cost Explorer

"Detect Anomalies"
→ ML detecta gastos anormais
→ Notifica via SNS
```

---

## 💰 ROI: Investimento em Economia

```yaml
Exemplo com RI de 3 anos:

Investimento inicial: $200 (All Upfront)
Economia/mês: $35
Payback: 6 meses
Economia total 3 anos: $1,055
ROI: 427%
```

---

## ✅ Checklist: Otimização de Custos

- [ ] Analisar instâncias subutilizadas com Compute Optimizer
- [ ] Comprar Reserved Instances para workloads estáveis
- [ ] Usar Spot para jobs não-críticos
- [ ] Encerrar instâncias obsoletas
- [ ] Liberar Elastic IPs não usados
- [ ] Monitorar EBS não utilisado
- [ ] Configurar Budget Alerts
- [ ] Revisar mensalmente com Cost Explorer

---

## 🔗 Próximos Passos

1. ➡️ **[Troubleshooting](07-troubleshooting.md)**
2. ➡️ **[Conceitos Fundamentais](01-conceitos-fundamentais.md)** (revisão)

---

**Última atualização**: Junho 2026  
**Autora**: Joana Iris  
**Licença**: MIT
