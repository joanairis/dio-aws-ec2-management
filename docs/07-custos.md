# 💰 Otimização de Custos em EC2

## Modelos de Preço

### On-Demand (Padrão)

```
Custo: Mais caro
Flexibilidade: Máxima
Comprometimento: Nenhum

Exemplo:
t2.micro = $0.0116/hora = ~$8.50/mês

Ideal para:
- Desenvolvimento
- Testes
- Workloads imprevisíveis
```

### Reserved Instances (RI)

```
Comprometimento: 1 ou 3 anos
Desconto: 
  - 1 ano: ~40% off
  - 3 anos: ~60% off
Flexibilidade: Média (pode vender no marketplace)

Exemplo para t2.micro:
- On-Demand: $0.0116/hora = ~$8.50/mês = ~$102/ano
- 1-year RI: ~$61/ano + small monthly
- 3-year RI: ~$32/ano + small monthly

Ideal para:
- Produção
- Aplicações de longa duração
- Workloads previsíveis
```

### Spot Instances

```
Desconto: Até 90% off on-demand
Risco: Pode ser interrompida se preço sobe
Flexibilidade: Baixa (mas rápida em reagir)

Exemplo:
- On-Demand: $0.0116/hora
- Spot: ~$0.0035/hora (70% desconto)

Ideal para:
- Batch jobs
- Análise de dados
- Processamento paralelo
- Workloads tolerantes a interrupção

NÃO ideal para:
- Banco de dados
- Produção crítica
```

### Savings Plans

```
Flexibilidade: Alta (muda tipo de instância)
Desconto: ~30-72% (dependendo do plano)
Comprometimento: 1 ou 3 anos

Tipos:
1. Compute Savings Plans
   - Desconto em qualquer tipo/tamanho
   - Maior flexibilidade

2. EC2 Instance Savings Plans
   - Desconto em familia específica
   - Menos flexível mas melhor desconto

Ideal para:
- Workloads com padrão de uso
- Múltiplas instâncias
- Multi-região
```

---

## Estratégia de Custo-Benefício

### Para Desenvolvimento

```yaml
1º Passo: Use Free Tier
  - t2.micro (elegível)
  - 30GB armazenamento grátis
  - Muitos serviços free

2º Passo: Use Spot para testes
  - Economiza ~70%
  - Rápido de provisionar/destruir

3º Passo: Shutdown automático
  - Para instâncias às 18h
  - Inicie às 8h
  - Economiza 50%+ em dev
```

### Para Produção

```yaml
1º Passo: Use Reserved Instances
  - Comprometimento de 1-3 anos
  - Desconto significativo
  - Previsibilidade de custos

2º Passo: Use Savings Plans
  - Flexibilidade entre tipos
  - Melhor se múltiplas instâncias

3º Passo: Use Spot para burst
  - Auto scaling com Spot
  - Combina com Reserved (base) + Spot (pico)

4º Passo: Otimize tipos
  - Usar tipos menores quando possível
  - Right-sizing baseado em utilização
```

---

## Reduzir Custos

### 1. Right-Sizing

```
Problem: Instâncias muito grandes que não usam tudo

Solução:
1. Monitore por 30 dias
   - CPU < 20%?
   - Memória < 50%?
2. Se sim: reduce para tipo menor
3. Teste aplicação
4. Guarde economia

Exemplo:
- t3.large ($0.0832/h) → t3.medium ($0.0416/h)
- Economia: 50% = ~$300/mês
```

### 2. Parar Instâncias Não Usadas

```
Find unused:
aws ec2 describe-instances \
  --query "Reservations[*].Instances[*].[InstanceId,InstanceType,LaunchTime]" \
  --output table

Para cada uma:
1. Verificar última atividade (CloudTrail)
2. Se > 30 dias: Stop ou Terminate

Economia: 100% da cobrança de compute
```

### 3. Usar EBS Otimizado Seletivamente

```
EBS-Optimized custa $0.005-0.01/hora extra

Valor só se:
- Aplicação I/O intensiva
- Precisão de performance
- Produção crítica

Para desenvolvimento: Desativar
```

### 4. Otimizar Armazenamento

```yaml
EBS Tipos:
  gp3: Recomendado (melhor preço/performance)
  gp2: Legado (mais caro)
  io1/io2: Apenas se IOPS críticos
  st1: Throughput grande (big data)
  sc1: Cold storage (backup)

Strategy:
- Altere gp2 → gp3: Economize 20%
- Remova volumes não usados
- Use S3 para arquivos grandes
```

### 5. Transferência de Dados

```
Custos de Data Transfer:
- EC2 → Internet: $0.09/GB
- EC2 → EC2 (mesma AZ): Grátis
- EC2 → S3: Grátis
- CloudFront: $0.085/GB

Optimização:
1. Coloque instâncias na mesma AZ
2. Use NAT Gateway (custo) vs NAT Instance
3. Use CloudFront para conteúdo estático
4. Comprima dados
```

---

## Monitorar Custos

### Cost Explorer

```
Console AWS:
1. Billing → Cost Explorer
2. Selecione período
3. Group by:
   - Service
   - Instance Type
   - Region
4. Analise padrões
```

### Billing Alerts

```
Console AWS:
1. Billing → Preferences
2. Ativar "Receive Billing Alerts"
3. Criar alarme em SNS
4. Defina threshold (ex: $50)

Benefício: Ser alertado antes de surpresa
```

### AWS Budgets

```
Console AWS:
1. AWS Budgets → Create Budget
2. Selecione tipo:
   - Monthly cost budget
   - Forecast-based
3. Configure limite (ex: $100/mês)
4. Adicione ações automáticas:
   - Enviar alerta
   - Parar instâncias
   - Terminar spots
```

---

## Calculadora de Custos

### AWS Pricing Calculator

```
Site: https://calculator.aws/

Usar para:
1. Estimar custo de arquitetura
2. Comparar cenários
3. Planejar orçamento
4. Justificar para gerentes

Exemplo:
- 2x t2.micro on-demand
- 100 GB EBS gp3
- 100 GB data transfer
= ~$40/mês
```

---

## Caso de Uso Real

### Antes (Sem Otimização)

```
4x t3.2xlarge on-demand
= $1.3344/hora = ~$980/mês

Problemas:
- CPU média: 15%
- Memória média: 25%
- Muito oversized
```

### Depois (Otimizado)

```
2x t3.medium (reserved, 1-year)
= $0.60/mês + $30/ano = ~$33/mês

2x Spot t3.medium (para burst)
= $0.025/hora (média) = ~$18/mês

Total: ~$51/mês
Economia: 95%! (~$930/mês)
```

---

## Checklist de Otimização

```
☐ Usar t2.micro para desenvolvimento
☐ Parar instâncias não usadas
☐ Right-sizing das instâncias
☐ Trocar on-demand por Reserved/Savings
☐ Usar Spot para non-critical
☐ Remover EBS volumes não utilizados
☐ Trocar gp2 → gp3 onde possível
☐ Usar NAT Instance (não NAT Gateway)
☐ CloudFront para conteúdo estático
☐ Monitorar com Cost Explorer
☐ Definir alertas de orçamento
☐ Avaliar Trusted Advisor
```

---

**Próximo:** [Troubleshooting e Resolução de Problemas](08-troubleshooting.md)