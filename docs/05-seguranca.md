# 🔒 Segurança em EC2

## Camadas de Segurança

```
┌─────────────────────────────────────────────┐
│ 1. AWS Account Level (IAM)                  │
├─────────────────────────────────────────────┤
│ 2. Network Level (VPC, NACLs)               │
├─────────────────────────────────────────────┤
│ 3. Subnet Level (NACLs)                     │
├─────────────────────────────────────────────┤
│ 4. Instance Level (Security Groups)         │
├─────────────────────────────────────────────┤
│ 5. OS Level (Firewall, SSH Config)          │
├─────────────────────────────────────────────┤
│ 6. Application Level (Autenticação)         │
└─────────────────────────────────────────────┘
```

---

## 1. Security Groups

### O que é?

Um firewall virtual que controla tráfego de entrada e saída em nível de instância.

### Características:

```yaml
Tipo: Stateful
  - Tráfego de resposta é automaticamente permitido
  - Se permite entrada, permite saída relacionada

Regras:
  - Inbound: Controla entrada
  - Outbound: Controla saída (padrão permite tudo)

Padrão: Nega tudo não explicitamente permitido
```

### Configurações Recomendadas

#### Para Aplicação Web:

```yaml
Inbound Rules:
  - HTTP (80): 0.0.0.0/0
  - HTTPS (443): 0.0.0.0/0
  - SSH (22): Seu IP específico

Outbound Rules:
  - All: 0.0.0.0/0 (padrão)
```

#### Para Banco de Dados (Backend):

```yaml
Inbound Rules:
  - MySQL (3306): SG do app server apenas
  - SSH (22): Admin IP específico

Outbound Rules:
  - All: 0.0.0.0/0 (padrão)
```

### Como Criar/Editar:

```
Console AWS:
1. EC2 → Security Groups
2. Clique "Create Security Group" ou edite existente
3. Adicione regras:
   - Type: SSH, HTTP, HTTPS, Custom
   - Protocol: TCP, UDP
   - Port Range: 22, 80, 443, etc
   - Source/Destination: IP, CIDR, SG ID

AWS CLI (adicionar regra):
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxxx \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/32
```

### ❌ Erros Comuns

```
❌ SSH para 0.0.0.0/0
   → Qualquer pessoa pode tentar acessar

❌ Banco de dados para 0.0.0.0/0
   → Exposto para internet

✅ SSH para seu IP específico
✅ Banco para SG do aplicativo
```

---

## 2. Pares de Chaves (Key Pairs)

### Boas Práticas

#### Guardar a Chave Privada:

```bash
# Permissão correta SEMPRE
chmod 400 ~/.ssh/minha-chave.pem

# Backup seguro
# - Criptografado
# - Não em repositório Git público
# - Não em email

# Onde guardar:
# - ~/.ssh/ (padrão)
# - Gerenciador de senhas
# - Cofre criptografado
```

#### Rotação de Chaves:

```
Recomendado:
- Gerar nova chave a cada 90 dias
- Testar a nova antes de descartar a antiga
- Manter backup das antigas por 30 dias
```

#### Se Chave for Comprometida:

```
1. Criar nova chave par
2. Adicionar chave pública à instância
3. Remover chave comprometida
4. Verificar logs de acesso (CloudTrail)
5. Auditar alterações na instância
```

### Adicionar Nova Chave a Instância Existente

```bash
# 1. Conectar com chave antiga
ssh -i ~/.ssh/antiga-chave.pem ec2-user@seu-ip

# 2. Adicionar nova chave pública
echo "sua-nova-chave-publica" >> ~/.ssh/authorized_keys

# 3. Remover chave antiga (opcional)
# Editar ~/.ssh/authorized_keys e remover linha antiga

# 4. Testar nova chave
# Em novo terminal:
ssh -i ~/.ssh/nova-chave.pem ec2-user@seu-ip
```

---

## 3. IAM Roles e Permissions

### O que é IAM Role?

Credenciais temporárias para a instância acessar outros serviços AWS sem armazenar chaves.

### Quando Usar:

```
✅ Instância precisa acessar S3
✅ Instância precisa de DynamoDB
✅ Instância precisa escrever logs CloudWatch

❌ Nunca coloque chaves de acesso na instância
❌ Nunca configure credenciais manuais
```

### Como Criar:

```
Console AWS:
1. IAM → Roles
2. Clique "Create role"
3. Selecione "EC2"
4. Adicione policies (ex: S3ReadOnly)
5. Nomeie a role
6. Clique "Create"

Depois, ao criar instância:
IAM instance profile → Selecione a role criada
```

### Exemplo: Role para S3

```
Criar role "EC2-S3-Access" com policy:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}

Então na aplicação:
# Usar SDK (boto3, SDK Java, etc)
# Não precisa configurar credenciais!
s3_client = boto3.client('s3')
```

---

## 4. Atualizações de Segurança

### Atualizar Sistema Operacional

#### Linux (Ubuntu/Debian):

```bash
# Ver atualizações disponíveis
apt-cache upgradable

# Atualizar lista de pacotes
sudo apt-get update

# Atualizar pacotes
sudo apt-get upgrade -y

# Atualizar tudo incluindo kernel (cuidado!)
sudo apt-get dist-upgrade -y

# Aplicar patches de segurança apenas
sudo apt-get install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

#### Linux (Amazon Linux 2 / CentOS / RHEL):

```bash
# Verificar updates
sudo yum check-update

# Aplicar atualizações
sudo yum update -y

# Apenas patches de segurança
sudo yum update --security -y
```

### Ativar Atualizações Automáticas

#### Ubuntu:

```bash
# Instalar ferramenta
sudo apt-get install -y unattended-upgrades

# Editar configuração
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades

# Descomente para reiniciar automaticamente:
# Unattended-Upgrade::Automatic-Reboot "true";
```

---

## 5. VPC e Subnets

### Arquitetura Recomendada

```
┌─────────────────────────────────────────────┐
│ VPC (10.0.0.0/16)                           │
├─────────────────────────────────────────────┤
│ Public Subnet (10.0.1.0/24)                 │
│   - Web Servers                             │
│   - Internet Gateway                        │
├─────────────────────────────────────────────┤
│ Private Subnet (10.0.2.0/24)                │
│   - Database Servers                        │
│   - NAT Gateway (acesso saída)              │
└─────────────────────────────────────────────┘
```

### Private vs Public Subnets

#### Public Subnet:

```yaml
Características:
  - Internet Gateway associado
  - Route table tem rota 0.0.0.0/0 → IGW
  - Instâncias têm IP público
  - Acessíveis da internet

Uso: Web servers, load balancers
```

#### Private Subnet:

```yaml
Características:
  - Sem Internet Gateway
  - Acesso saída via NAT Gateway
  - Instâncias sem IP público
  - Não acessíveis da internet diretamente

Uso: Banco de dados, cache, backend
```

---

## 6. Monitoramento de Segurança

### CloudTrail

```
O que registra:
- Quem fez o quê
- Quando foi feito
- De onde foi feito
- Resultado da ação

Como usar:
1. CloudTrail → Create Trail
2. Escolha bucket S3 para logs
3. Selecione eventos para rastrear
4. Ative trail

Exemplo de evento:
{
  "eventName": "RunInstances",
  "eventTime": "2026-05-27T17:54:03Z",
  "userIdentity": {
    "principalId": "AIDAJ45Q7YFFAREXAMPLE",
    "type": "IAMUser"
  }
}
```

### VPC Flow Logs

```
O que registra:
- Tráfego aceito/rejeitado
- IPs de origem/destino
- Portas usadas
- Protocolos

Como usar:
1. VPC → Flow Logs
2. Selecione VPC ou subnet
3. Escolha CloudWatch Logs ou S3
4. Crie IAM role
5. Ative logs

Benefícios:
- Diagnosticar problemas de conectividade
- Monitorar tráfego
- Auditoria de segurança
```

---

## 7. Checklista de Segurança

### ✅ Antes de Colocar em Produção

```
Network:
☐ VPC configurada corretamente
☐ Subnets públicas/privadas isoladas
☐ NAT Gateway para saída de tráfego
☐ NACLs restritivos (quando necessário)

Security Groups:
☐ SSH apenas de IP específico
☐ HTTP/HTTPS de 0.0.0.0/0 (se web app)
☐ Banco de dados apenas de app SG
☐ Sem acesso desnecessário

Instância:
☐ Sistema operacional atualizado
☐ Software atualizado
☐ Firewall do SO configurado
☐ Serviços desnecessários desativados

Credenciais:
☐ Sem chaves de acesso na instância
☐ IAM Role com permissões mínimas
☐ Par de chaves seguro
☐ Nenhuma credencial em texto plano

Monitoring:
☐ CloudTrail ativado
☐ CloudWatch alarmes configurados
☐ VPC Flow Logs ativados
☐ Logs centralizados

Backup:
☐ Snapshots agendados
☐ AMI custom criada
☐ Backup em local seguro
```

---

## 8. Respostas a Incidentes

### Instância Comprometida - O Que Fazer:

```
1. ISOLAR:
   - Remova Elastic IP
   - Mude para Security Group restritivo
   - NÃO encerre ainda

2. INVESTIGAR:
   - Ative VPC Flow Logs
   - Verifique CloudTrail
   - Revise histórico de conexões SSH
   - Copie logs do sistema

3. REMEDIAR:
   - Crie AMI para análise forense
   - Encerre instância comprometida
   - Crie nova instância limpa
   - Reaponte DNS/Load Balancer

4. PREVENIR:
   - Atualize segurança grupos
   - Rotacione chaves
   - Revise IAM permissions
   - Implemente MFA
```

---

**Próximo:** [Monitoramento e Otimização](06-monitoramento.md)
