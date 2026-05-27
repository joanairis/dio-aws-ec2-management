# ⚙️ Gerenciando Instâncias EC2

## Operações Principais

### 1. Parar Instância (Stop)

#### O que Acontece:

- Instância entra em estado "stopping" → "stopped"
- Dados são preservados (EBS)
- NÃO é cobrada por computação
- MAS é cobrada por armazenamento EBS
- Leva ~1-3 minutos

#### Quando Usar:

```
✅ Desenvolvimento temporário
✅ Testes que serão retomados
✅ Reduzir custos de curta duração
❌ Não para parar permanentemente (use Terminate)
```

#### Como Fazer:

```
Console AWS:
1. Ir para EC2 → Instances
2. Clicar com botão direito na instância
3. Instance State → Stop
4. Confirmar

AWS CLI:
aws ec2 stop-instances --instance-ids i-xxxxxxxxx
```

---

### 2. Iniciar Instância (Start)

#### O que Acontece:

- Instância retorna ao estado "running"
- IP privado permanece o mesmo
- IP público PODE mudar (use Elastic IP se for problema)
- Inicia cobrança novamente
- Leva ~1-3 minutos

#### Como Fazer:

```
Console AWS:
1. Ir para EC2 → Instances
2. Clicar com botão direito na instância
3. Instance State → Start
4. Confirmar

AWS CLI:
aws ec2 start-instances --instance-ids i-xxxxxxxxx
```

---

### 3. Reiniciar Instância (Reboot)

#### O que Acontece:

- Instância permanece "running"
- IP público não muda
- Dados locais são preservados
- Leva ~1 minuto
- Como reiniciar um computador

#### Quando Usar:

```
✅ Aplicar patches
✅ Atualizar kernel
✅ Resolver problemas de software
❌ Terminar a instância (use Stop ou Terminate)
```

#### Como Fazer:

```
Console AWS:
1. Ir para EC2 → Instances
2. Clicar com botão direito na instância
3. Instance State → Reboot
4. Confirmar

AWS CLI:
aws ec2 reboot-instances --instance-ids i-xxxxxxxxx
```

---

### 4. Encerrar Instância (Terminate)

#### ⚠️ CUIDADO: AÇÃO IRREVERSÍVEL!

#### O que Acontece:

- Instância entra em estado "shutting-down" → "terminated"
- Todos os dados locais são deletados
- Volumes EBS com "Delete on Termination" são deletados
- Cobrança para quando encerrada
- Não pode ser revertido

#### Quando Usar:

```
✅ Aplicação não é mais necessária
✅ Limpeza de ambiente
✅ Reduzir custos permanentemente
❌ Parada temporária (use Stop)
```

#### Como Fazer:

```
Console AWS:
1. Ir para EC2 → Instances
2. Clicar com botão direito na instância
3. Instance State → Terminate
4. CONFIRMAR (aviso de risco!)

AWS CLI:
aws ec2 terminate-instances --instance-ids i-xxxxxxxxx
```

---

## Gerenciar IPs Públicos

### Problema: IP Público Muda ao Parar/Iniciar

```
Comportamento:
- Parada: IP público é liberar
- Iniciada: Novo IP público é atribuído
- Problema: Scripts e DNS configs quebram
```

### Solução: Elastic IP

#### O que é?

Um IP público estático que não muda.

#### Como Alocar:

```
Console AWS:
1. EC2 → Elastic IPs
2. Clique "Allocate Elastic IP"
3. Escolha região
4. Clique "Allocate"

AWS CLI:
aws ec2 allocate-address --domain vpc
```

#### Como Associar à Instância:

```
Console AWS:
1. EC2 → Elastic IPs
2. Clique no IP alocado
3. Clique "Associate Elastic IP"
4. Selecione sua instância
5. Clique "Associate"

AWS CLI:
aws ec2 associate-address --instance-id i-xxxxxxxxx \
  --allocation-id eipalloc-xxxxxxxx
```

#### Custos:

```
GRÁTIS: Elastic IP associado a instância running
PAGO: $0.005/hora se alocado mas não associado
```

#### ✅ Dica:

Para desenvolvimento com free tier:
1. Aloque Elastic IP
2. Associe à instância
3. Use esse IP em todos os scripts
4. Parar/iniciar não afeta o IP

---

## Modificar Instâncias

### 1. Redimensionar (Change Instance Type)

#### Quando Usar:

```
❌ Não é possível ao vivo (instância running)
✅ Parar a instância
✅ Alterar tipo
✅ Iniciar de novo
```

#### Limitações:

```
Não pode redimensionar entre:
- General Purpose ↔ Compute Optimized (possível)
- Virtualization Type diferente
- Arquitetura diferente (x86 ↔ ARM)
```

#### Como Fazer:

```
Console AWS:
1. Parar a instância
2. Clique com botão direito
3. Instance Settings → Change Instance Type
4. Selecione novo tipo
5. Clique "Apply"
6. Iniciar novamente
```

---

### 2. Modificar Security Groups

#### Como Fazer:

```
Console AWS:
1. Ir para EC2 → Instances
2. Selecione instância
3. Em "Security" → "Security groups"
4. Clique no grupo
5. Edite as regras necessárias

AWS CLI (adicionar regra):
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 3306 \
  --cidr 10.0.0.0/8
```

---

### 3. Adicionar/Remover Tags

#### Como Fazer:

```
Console AWS:
1. Ir para EC2 → Instances
2. Selecione instância
3. Aba "Tags"
4. Clique "Add tag"
5. Preencha Key e Value

AWS CLI:
aws ec2 create-tags \
  --resources i-xxxxxxxxx \
  --tags Key=Environment,Value=Production
```

---

## Snapshots e Backups

### 1. Criar Snapshot de EBS

#### O que é?

Um backup ponto-em-tempo do volume EBS.

#### Quando Usar:

```
✅ Antes de alterações importantes
✅ Backup periódico
✅ Clonar volumes
✅ Compartilhar entre regiões
```

#### Como Fazer:

```
Console AWS:
1. EC2 → Volumes
2. Clique no volume
3. Clique com botão direito
4. Create Snapshot
5. Adicione descrição
6. Clique "Create snapshot"

AWS CLI:
aws ec2 create-snapshot \
  --volume-id vol-xxxxxxxxx \
  --description "Backup da minha app"
```

---

### 2. Criar AMI da Instância

#### O que é?

Uma imagem da instância que pode ser usada para clonar.

#### Quando Usar:

```
✅ Golden image de aplicação
✅ Replicar configuração
✅ Disaster recovery
✅ Auto-scaling
```

#### Como Fazer:

```
Console AWS:
1. EC2 → Instances
2. Clique com botão direito
3. Image and templates → Create image
4. Preencha nome e descrição
5. Clique "Create image"

AWS CLI:
aws ec2 create-image \
  --instance-id i-xxxxxxxxx \
  --name "minha-app-v1" \
  --description "Aplicação pronta"
```

---

## Monitoramento Básico

### 1. Status Checks

```
System Status Check:
  - Hardware/rede AWS
  - Se falhar: para e inicia (novo hardware)

Instance Status Check:
  - SO e aplicação
  - Se falhar: reinicia sistema operacional
```

#### Como Verificar:

```
Console AWS:
1. EC2 → Instances
2. Selecione instância
3. Aba "Status checks"
4. Verifique status (deve ser "2/2 passed")
```

---

### 2. CloudWatch Metrics

#### Métricas Principais:

```
CPU Utilization: % de CPU em uso (0-100%)
Network In: Dados recebidos (bytes)
Network Out: Dados enviados (bytes)
Disk Read Ops: Operações de leitura
Disk Write Ops: Operações de escrita
Status Check Failed: Falha em check
```

#### Como Acessar:

```
Console AWS:
1. EC2 → Instances
2. Selecione instância
3. Aba "Monitoring"
4. Veja gráficos de métrica

Ou:

CloudWatch → Dashboards → Criar dashboard customizado
```

---

### 3. Criar Alarme

#### Como Fazer:

```
Console AWS:
1. CloudWatch → Alarms → Create alarm
2. Selecione métrica (ex: CPU Utilization)
3. Configure threshold (ex: > 80%)
4. Configure notificação (SNS topic)
5. Clique "Create alarm"

AWS CLI:
aws cloudwatch put-metric-alarm \
  --alarm-name "high-cpu" \
  --alarm-description "CPU > 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold
```

---

## Operações em Lote (CLI)

### Parar Todas as Instâncias com Tag

```bash
aws ec2 stop-instances \
  --filters "Name=tag:Environment,Values=Development" \
  --region us-east-1
```

### Listar Todas as Instâncias

```bash
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress,Tags[0].Value]" \
  --output table
```

### Terminar Instâncias por Tag

```bash
aws ec2 terminate-instances \
  --filters "Name=tag:Temporary,Values=true" \
  --region us-east-1
```

---

## Resumo de Estados

| Estado | Cobrança | Dados | Ação Necessária |
|--------|----------|-------|-----------------|
| running | ✅ CPU + EBS | ✅ Presente | Normal |
| stopped | ❌ CPU | ✅ Presente | Pode iniciar |
| terminated | ❌ CPU | ❌ Perdido | Irreversível |
| stopping | ✅ Transição | ✅ Presente | Aguardar |

---

**Próximo:** [Boas Práticas e Segurança](05-seguranca.md)
