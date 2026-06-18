# 📊 Monitoramento e Logs em EC2

Guia completo para monitorar o desempenho e saúde de suas instâncias EC2.

---

## 🎯 CloudWatch: Monitoramento Central

**CloudWatch** é o serviço de monitoramento da AWS que coleta métricas, logs e eventos.

### Ativar Monitoramento Detalhado

```
Console EC2 → Instância → Ações → Monitoramento
→ "Ativar monitoramento detalhado"

Custos: ~$0.35 por métrica/mês
Benefício: Dados a cada 1 minuto (em vez de 5 minutos)
```

---

## 📈 Métricas Principais

### CPU Utilization

```yaml
O que é: Percentual de CPU em uso
Normal: 5-50%
Alerta: > 80% por mais de 5 minutos
Ação: Scale up ou otimizar aplicação
```

**Gráfico esperado**:
```
100% ┤
     │     ╭─╮
 80% ┤    ╭─╯ ╰─╮
 60% ┤   ╭╯     ╰╮
 40% ┤  ╭╯       ╰╮
 20% ┤─╭╯         ╰─
  0% ┴─┴───────────────
     0  5  10 15 20
```

### Network In/Out

```yaml
O que é: Dados recebidos/enviados (bytes)
Normal: Varia muito conforme aplicação
Alerta: Picos repentinos ou quedas
Ação: Investigar tráfego anormal
```

### Disk Read/Write

```yaml
O que é: Operações de disco (por segundo)
Normal: < 1000 ops/sec
Alerta: > 5000 ops/sec consistente
Ação: Verificar I/O de banco de dados ou arquivos
```

### Status Checks

```yaml
System Status:
  ✅ OK: Hardware está funcionando
  ❌ Falha: Problema com host, rede ou alimentação

Instance Status:
  ✅ OK: Sistema operacional e aplicação funcionam
  ❌ Falha: Problema com SO, kernel ou aplicação
```

---

## 📊 Ver Métricas no Console

### Dashboard Padrão

```
1. Console EC2 → Instância
2. Aba "Monitoramento"
3. Veja últimas 1-24 horas de:
   - CPU Utilization
   - Network In/Out
   - Disk Operations
   - Status Checks
```

### CloudWatch Dashboard Customizado

```
1. AWS Console → CloudWatch → Dashboards
2. "Criar dashboard"
3. Nome: "Produção-EC2-Dashboard"
4. Adicione widgets:
   - CPU por instância
   - Memória (com CloudWatch Agent)
   - Disco (com CloudWatch Agent)
   - Network
```

---

## 🚨 Alarmes CloudWatch

### Criar Alarme de CPU Alta

```
1. Console EC2 → Instância → Monitoramento
2. Gráfico "CPU Utilization" → "Criar alarme"
3. Configurar:
   - Estatística: Average
   - Período: 5 minutos
   - Limiar: 80%
   - Ação: Enviar notificação SNS
```

### Criar Alarme de Instância Parada

```
1. Console CloudWatch → Alarmes
2. "Criar alarme"
3. Métrica: EC2 → InstanceState
4. Configurar:
   - Condição: = 80 (stopped)
   - Ação: Notificar via email
```

### Notificações via SNS

```
1. Console SNS → Tópicos
2. "Criar tópico"
3. Nome: ec2-alerts
4. "Criar assinatura"
5. Protocolo: Email
6. Endpoint: seu-email@example.com
7. Confirmar via email recebido
```

---

## 📝 Logs com CloudWatch Logs

### Instalar CloudWatch Agent

```bash
# Download (Ubuntu)
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb

# Instalar
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb

# Configurar
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

# Iniciar
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
    -a fetch-config \
    -m ec2 \
    -s
```

### Enviar Logs de Aplicação

```bash
# Exemplo: Enviar logs de aplicação
# Arquivo: /var/log/app.log → CloudWatch Logs

# Editar config
sudo nano /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json

# Adicionar:
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/app.log",
            "log_group_name": "/aws/ec2/app",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
```

### Ver Logs no Console

```
AWS Console → CloudWatch → Log Groups
Procure: /aws/ec2/app
Clique em log stream para ver eventos
```

---

## 🔍 Logs do Sistema

### EC2 System Log

```
Console EC2 → Instância → Status checks
→ Clique em "System log"

Ver:
- Boot messages
- Hardware issues
- Network configuration
```

### EC2 Instance Log

```
Console EC2 → Instância → Status checks
→ Clique em "Instance log"

Ver:
- Startup scripts
- Application logs
- Kernel panic messages
```

---

## 📈 CloudWatch Insights (Análise de Logs)

### Criar Query de Logs

```sql
-- Contar erros por hora
fields @timestamp, @message
| stats count() as error_count by bin(5m)

-- Buscar erros de autenticação
fields @timestamp, @message
| filter @message like /failed/
| stats count() as auth_failures
```

### Executar Query

```
1. CloudWatch → Log Groups
2. "Insights"
3. Cole query acima
4. Clique "Executar query"
5. Veja gráfico de erros
```

---

## 🔗 Integração com Alertas

### Auto Scaling com Alarmes

```yaml
Auto Scaling Group:
  Min: 1
  Max: 5
  Desejado: 2

Política de Scale Up:
  Trigger: CPU > 70% por 2 minutos
  Ação: Adicione 1 instância

Política de Scale Down:
  Trigger: CPU < 30% por 5 minutos
  Ação: Remova 1 instância
```

### SNS + Lambda

```yaml
CloudWatch Alarm (CPU alta)
    ↓
SNS Topic (ec2-alerts)
    ↓
Lambda Function (escalona)
    ↓
Auto Scaling: +1 instância
```

---

## 📊 Relatórios e Análise

### CloudWatch Metrics Math

```
Criar métrica customizada:

m1 = (CPUUtilization) / 2 + (NetworkIn) / 100000
(Fórmula customizada para seu caso de uso)
```

### AWS Cost Explorer

```
1. AWS Console → Cost Explorer
2. Filtrar: EC2 instances
3. Ver custos por:
   - Tipo de instância
   - Região
   - Período
```

---

## 🚨 Status Checks: Interpretação

### ✅ 2/2 checks passed

```
Significado: Tudo OK!
Ação: Nenhuma
```

### ❌ 1/2 checks passed

```
Significado: System Status FALHOU
Causa: Hardware, rede, alimentação
Ação: Parar e iniciar (novo hardware)
```

### ❌ 0/2 checks passed

```
Significado: AMBOS falharam
Causa: Grave problema
Ação: 
1. Parar instância
2. Iniciar em novo host
3. Se persistir: contate suporte AWS
```

---

## 📋 Checklist: Monitoramento

- [ ] Ativar monitoramento detalhado
- [ ] Criar alarmes para CPU, Disco, Network
- [ ] Configurar SNS para notificações
- [ ] Instalar CloudWatch Agent
- [ ] Coletar logs de aplicação
- [ ] Revisar logs regularmente
- [ ] Configurar Auto Scaling
- [ ] Criar dashboard customizado

---

## 🔗 Próximos Passos

1. ➡️ **[Otimização de Custos](06-custos.md)**
2. ➡️ **[Troubleshooting](07-troubleshooting.md)**

---

**Última atualização**: Junho 2026  
**Autora**: Joana Iris  
**Licença**: MIT
