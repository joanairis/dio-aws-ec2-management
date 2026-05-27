# 📊 Monitoramento e Otimização de Performance

## CloudWatch - Visão Geral

CloudWatch é o serviço de monitoramento da AWS que fornece métricas, logs e alarmes.

### Métricas Padrão do EC2

```
CPU Utilization
  - % de CPU em uso
  - Padrão: 5 minutos
  - Importante para: Auto-scaling, alertas

Network In
  - Dados recebidos (bytes)
  - Importante para: Análise de tráfego

Network Out
  - Dados enviados (bytes)
  - Importante para: Transferência de dados

Disk Read Operations
  - Leituras de disco por segundo
  - Importante para: Gargalos de I/O

Disk Write Operations
  - Escritas de disco por segundo
  - Importante para: Gargalos de I/O

Status Check Failed
  - Falha em health checks
  - Importante para: Disponibilidade
```

### Acessar Métricas

```
Console AWS:
1. EC2 → Instances
2. Selecione instância
3. Aba "Monitoring"
4. Veja gráficos das últimas horas

Ou:

CloudWatch → Dashboards → Criar dashboard personalizado
```

---

## Configurar Alarmes

### Alarme para CPU Alta

```
Console AWS:
1. CloudWatch → Alarms → Create alarm
2. Select metric:
   - Namespace: AWS/EC2
   - Metric name: CPUUtilization
   - Statistic: Average
   - Period: 5 minutes
3. Set threshold:
   - Condition: Greater than
   - Value: 80
4. Configure actions:
   - Send notification to SNS topic
   - Topic: criar nova ou selecionar
   - Email ou Slack
5. Create alarm
```

### Alarme para Espaço em Disco

```
⚠️ Nota: EC2 não envia métrica de disco por padrão

Solução: Instalar CloudWatch Agent

1. Criar IAM role com CloudWatchAgentServerPolicy
2. Anexar role à instância
3. SSH na instância:
   wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
   sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
4. Configurar agent:
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
5. Iniciar agent:
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
     -a fetch-config \
     -m ec2 \
     -s
6. Criar alarme para DisckUsed
```

---

## CloudWatch Logs

### Por que Usar?

```
✅ Centralizar logs de aplicação
✅ Pesquisar e analisar
✅ Criar alarmes baseados em padrões
✅ Reter logs por longos períodos
✅ Integrar com outras ferramentas
```

### Instalar CloudWatch Logs Agent

```bash
# 1. Criar IAM role com CloudWatchLogsAgentServerPolicy

# 2. SSH na instância
ssh -i ~/.ssh/chave.pem ubuntu@seu-ip

# 3. Instalar agent
wget https://s3.amazonaws.com/aws-cloudwatch/downloads/latest/awslogs-agent-setup.py
sudo python3 awslogs-agent-setup.py -n

# 4. Configurar log groups
# Edite: /var/awslogs/etc/awslogs.conf
[/var/log/syslog]
log_group_name = /aws/ec2/syslog
log_stream_name = {instance_id}
file = /var/log/syslog
datetime_format = %b %d %H:%M:%S

# 5. Iniciar serviço
sudo service awslogs start

# 6. Verificar status
sudo service awslogs status
```

### Criar Metric Filter

```
Console AWS:
1. CloudWatch → Log Groups
2. Selecione log group
3. Aba "Metric Filters"
4. "Create Metric Filter"
5. Padrão de busca:
   [ERROR]
6. Test Pattern
7. Crie métrica customizada
8. Crie alarme baseado nessa métrica
```

---

## Otimizar Performance

### 1. Escolher o Tipo Correto de Instância

```yaml
Analisar:
  - CPU: Picos acima de 80%? Aumentar tipo
  - Memória: Usar CloudWatch agent para medir
  - I/O: Muitos disco ops? Usar io1/io2
  - Network: Alto throughput? Usar enhanced networking

Ferramentas:
  - aws ec2 describe-instance-types
  - AWS Compute Optimizer (analisa padrão)
  - Trusted Advisor (recomendações)
```

### 2. Usar EBS Otimizado

```
Benefício: Dedicar banda de rede para EBS
Custo: +$0.005/hora (mínimo)

Console AWS:
1. Parar instância
2. Clique botão direito
3. Instance Settings → EBS-Optimized
4. Ativar
5. Iniciar

AWS CLI:
aws ec2 modify-instance-attribute \
  --instance-id i-xxxxxxxxx \
  --ebs-optimized
```

### 3. Enhanced Networking

```
Para alta performance de rede (10 Gbps+)

AWS CLI:
aws ec2 modify-instance-attribute \
  --instance-id i-xxxxxxxxx \
  --sriov-net-support simple
```

### 4. Usar Auto Scaling

```
Adicionar/remover instâncias automaticamente

Console AWS:
1. EC2 → Auto Scaling Groups
2. Create Auto Scaling Group
3. Selecione launch template/configuration
4. Configure policies:
   - Scale Up se CPU > 70%
   - Scale Down se CPU < 30%
5. Defina min/max/desired capacity
6. Create
```

---

## Identificar Gargalos

### Alto Uso de CPU

```
Causas possíveis:
- Aplicação computacionalmente intensiva
- Vazamento de memória → swap → CPU
- Muitos processos concorrentes
- Código ineficiente

Solução:
1. SSH na instância
2. htop para ver processos
3. Identificar processo culpado
4. Otimizar código ou aumentar tipo
```

### Alto Uso de Memória

```
Causas possíveis:
- Cache não limitado
- Vazamento de memória
- Muitas conexões abertas
- Tipo de instância insuficiente

Solução:
1. ssh na instância
2. free -h para ver memória
3. Instalar CloudWatch agent para métricas
4. Aumentar RAM ou otimizar aplicação
```

### Alto I/O de Disco

```
Causas possíveis:
- Muitos writes em banco de dados
- Operações de arquivo grandes
- Falta de índices no banco

Solução:
1. Trocar EBS gp3 → io1/io2
2. Aumentar IOPS
3. Otimizar queries
4. Usar SSD cache
```

---

## Troubleshooting Comum

### Instância Lenta

```
Passo 1: Verificar CPU
  - CloudWatch → CPU Utilization
  - Se > 80%: upgrade tipo ou otimizar app

Passo 2: Verificar Memória
  - SSH na instância
  - free -h
  - Se pouca: aumentar RAM

Passo 3: Verificar Disco
  - df -h
  - Se cheio: liberar espaço ou aumentar volume

Passo 4: Verificar I/O
  - iostat -x 1 5
  - Se alto: melhorar queries ou tipo EBS
```

### Latência Alta

```
Causas:
- Network latency
- Instância noutra AZ
- Security group restritivo

Solução:
- Usar VPC Peering para multi-AZ
- Coloque em mesma AZ
- Usar placement groups
```

---

## Status Checks

### System Status Check

```
Verifica:
- Hardware físico
- Conectividade de rede
- Alimentação

Se falhar:
  1. Parar instância
  2. Aguardar 5 minutos
  3. Iniciar (novo hardware será alocado)
  4. Se ainda falhar: contatar AWS Support
```

### Instance Status Check

```
Verifica:
- Sistema Operacional
- Drivers
- Kernel
- Aplicação

Se falhar:
  1. Tentar reboot
  2. Se não funcionar, verificar logs do SO
  3. Último recurso: recriar de AMI
```

---

## Dashboards Personalizados

### Criar Dashboard

```
Console AWS:
1. CloudWatch → Dashboards
2. Create dashboard
3. Nomeie
4. Add widgets:
   - CPU graph
   - Memory graph
   - Network graph
   - Custom metrics
5. Save
```

### Exemplo de Dashboard

```json
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["AWS/EC2", "CPUUtilization", {"stat": "Average"}],
          [".", "NetworkIn", {"stat": "Sum"}],
          [".", "NetworkOut", {"stat": "Sum"}]
        ],
        "period": 300,
        "stat": "Average",
        "region": "us-east-1",
        "title": "EC2 Performance"
      }
    }
  ]
}
```

---

**Próximo:** [Otimização de Custos](07-custos.md)