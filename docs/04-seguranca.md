# 🔐 Práticas de Segurança em EC2

Guia essencial para manter suas instâncias EC2 seguras.

---

## 🚨 Princípios Fundamentais

### Princípio do Menor Privilégio

```
❌ ERRADO:
SSH (22): 0.0.0.0/0 ← Aberto para toda internet!

✅ CORRETO:
SSH (22): 203.0.113.45/32 ← Apenas seu IP
```

### Defesa em Profundidade

```
Camada 1: Security Groups (firewall)
    ↓
Camada 2: Network ACLs (subnet firewall)
    ↓
Camada 3: Firewall no SO (iptables/ufw)
    ↓
Camada 4: Autenticação em aplicação
    ↓
Camada 5: Criptografia de dados em trânsito
```

---

## 🔓 Grupos de Segurança

### Configuração Básica Segura

#### Web Server (HTTP/HTTPS)

```yaml
Entrada (Inbound):
  - SSH (22): De seu-ip/32
  - HTTP (80): De 0.0.0.0/0
  - HTTPS (443): De 0.0.0.0/0

Saída (Outbound):
  - Todos os protocolos: Para 0.0.0.0/0 (padrão)
```

#### Banco de Dados (Privado)

```yaml
Entrada (Inbound):
  - MySQL (3306): De sg-web-server (reference)
  - SSH (22): De seu-ip/32

Saída (Outbound):
  - Todos os protocolos: Para 0.0.0.0/0
```

#### API Backend

```yaml
Entrada (Inbound):
  - SSH (22): De seu-ip/32
  - HTTP (80): De sg-load-balancer (reference)
  - HTTPS (443): De sg-load-balancer (reference)
  - Custom TCP (3000): De sg-load-balancer

Saída (Outbound):
  - Todos os protocolos: Para 0.0.0.0/0
```

### Como Criar Regra Segura

```
1. Em vez de usar CIDR (0.0.0.0/0), use:
   ✅ Seu IP específico: X.X.X.X/32
   ✅ Referência a outro SG: sg-xxxxxxxx
   ✅ Intervalo corporativo: 10.0.0.0/8

2. Nunca abra SSH (22) para 0.0.0.0/0
   ✅ Use VPN se acesso remoto
   ✅ Use AWS Systems Manager Session Manager

3. Use Security Groups como firewall:
   ✅ DB só aceita de web server
   ✅ Web server só aceita de load balancer
```

### Referência Entre Security Groups

```
SG-Load-Balancer
  └─ Aceita: HTTP/HTTPS de 0.0.0.0/0

SG-Web-Server
  └─ Aceita: HTTP de SG-Load-Balancer ← Referência!

SG-Database
  └─ Aceita: MySQL de SG-Web-Server ← Referência!
```

**Vantagem**: Automático! Não precisa atualizar IPs

---

## 🔑 Gerenciamento de Chaves SSH

### Armazenar Chaves com Segurança

```bash
# Linux/Mac: Arquivo home directory
mkdir -p ~/.ssh
mv minha-chave.pem ~/.ssh/
chmod 600 ~/.ssh/minha-chave.pem
chmod 700 ~/.ssh

# Windows: Documentos/Chaves AWS
# C:\Users\Usuario\Documents\AWS-Keys\
# Proteger: Propriedades → Segurança → Apenas você
```

### Rotação de Chaves

```
Recomendação: A cada 90 dias

Passos:
1. Criar novo par de chaves
2. Adicionar chave pública ao ~/.ssh/authorized_keys
3. Testar conexão com nova chave
4. Remover chave antiga
5. Documentar mudança
```

### Usar AWS Secrets Manager

```bash
# Armazenar chave em Secrets Manager em vez de arquivo
aws secretsmanager create-secret \
  --name ec2-prod-key \
  --secret-string file://minha-chave.pem
```

### Usar AWS Systems Manager Parameter Store

```bash
# Armazenar chave pública
aws ssm put-parameter \
  --name /ec2/ssh/public-key \
  --value "$(cat ~/.ssh/minha-chave.pub)" \
  --type String
```

---

## 👤 Acesso e Autenticação

### Usar IAM Roles (Em vez de Access Keys)

```yaml
❌ ERRADO (em instância EC2):
# Salvar credentials no arquivo
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

✅ CORRETO:
# Usar IAM Role associada à instância
# Credenciais vêm automaticamente via metadata
```

### Criar e Associar IAM Role

```bash
# 1. Criar role
aws iam create-role \
  --role-name EC2-S3-Access \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "Service": "ec2.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }'

# 2. Associar política
aws iam attach-role-policy \
  --role-name EC2-S3-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# 3. Criar instance profile
aws iam create-instance-profile \
  --instance-profile-name EC2-S3-Profile

# 4. Adicionar role ao profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-Profile \
  --role-name EC2-S3-Access
```

### Multi-Factor Authentication (MFA)

```bash
# Exigir MFA para ações sensíveis
aws iam enable-mfa-device \
  --user-name seu-usuario \
  --serial-number arn:aws:iam::123456789:mfa/seu-usuario \
  --authentication-code1 123456 \
  --authentication-code2 789012
```

---

## 🛡️ Atualizações e Patches

### Ubuntu/Debian

```bash
# Conectar à instância
ssh -i minha-chave.pem ubuntu@seu-ip

# Atualizar lista de pacotes
sudo apt-get update

# Atualizar sistema
sudo apt-get upgrade -y

# Atualizar kernel (requer reinicialização)
sudo apt-get install --only-upgrade linux-image-generic

# Rebootar se necessário
sudo reboot
```

### Amazon Linux 2

```bash
# Conectar à instância
ssh -i minha-chave.pem ec2-user@seu-ip

# Atualizar sistema
sudo yum update -y

# Rebootar
sudo reboot
```

### Windows

```powershell
# Usar Windows Update ou:
# Settings → Update & Security → Check for updates
```

### Automatizar Atualizações

```yaml
# Usar UserData script na criação:
#!/bin/bash
apt-get update
apt-get upgrade -y
apt-get install -y unattended-upgrades
systemctl enable unattended-upgrades
```

---

## 🔒 Encriptação

### Encriptação em Repouso (EBS)

```
Console EC2 → Volumes → Criar volume
  └─ Criptografia: ✅ Ativar

Algoritmo: AWS managed key (padrão)
Custo: Sem taxa adicional
```

### Encriptação em Trânsito (SSL/TLS)

```bash
# Usar HTTPS em vez de HTTP
# Instalar certificado SSL/TLS

# Com Let's Encrypt (gratuito)
sudo apt-get install certbot python3-certbot-nginx

# Com AWS Certificate Manager (gratuito para AWS)
# Recomendado para load balancers
```

### Encriptação de Dados Sensíveis

```bash
# Instalar encriptação de filesystem
sudo apt-get install ecryptfs-utils

# Usar para pastas sensíveis
mount -t ecryptfs /dados /dados
```

---

## 🚪 Acesso Remoto Seguro

### AWS Systems Manager Session Manager

**Vantagem**: Sem SSH, sem abrir portas

```bash
# 1. Instância precisa ter IAM Role com AmazonSSMManagedInstanceCore
# 2. Instância precisa de Systems Manager Agent (padrão em muitas AMIs)

# 3. Conectar via CLI
aws ssm start-session --target i-0123456789abcdef0

# 4. Saída
sh-4.2$
```

### VPN para Acesso SSH

```
Alternativa ao Systems Manager:

1. Configurar OpenVPN na AWS
2. Conectar ao VPN
3. Usar SSH para instância privada
4. Benefício: Tráfego criptografado
```

### AWS Session Manager com Port Forwarding

```bash
# Forwarding de porta SSH (222 local → 22 remoto)
aws ssm start-session \
  --target i-0123456789abcdef0 \
  --document-name AWS-StartPortForwardingSession \
  --parameters "portNumber=22,localPortNumber=222"

# Conectar
ssh -i minha-chave.pem -p 222 ubuntu@localhost
```

---

## 📊 Monitoramento de Segurança

### CloudTrail (Auditoria)

```bash
# Ativar CloudTrail
aws cloudtrail create-trail \
  --name my-trail \
  --s3-bucket-name my-bucket

# Iniciar logging
aws cloudtrail start-logging --trail-name my-trail

# Ver eventos
aws cloudtrail lookup-events --trail-name my-trail
```

### VPC Flow Logs

```bash
# Registrar tráfego de rede
aws ec2 create-flow-logs \
  --resource-type NetworkInterface \
  --resource-ids eni-xxxxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs
```

### AWS GuardDuty (Detecção de Ameaças)

```bash
# Ativar GuardDuty
aws guardduty create-detector \
  --enable

# Ver achados
aws guardduty list-findings \
  --detector-id xxxxx
```

---

## ✅ Checklist de Segurança

- [ ] Security Groups restritos (não use 0.0.0.0/0 para SSH)
- [ ] Chaves SSH armazenadas com segurança (chmod 600)
- [ ] IAM Roles em vez de Access Keys
- [ ] Sistema atualizado (apt-get update && upgrade)
- [ ] Firewall no SO ativo (ufw/iptables)
- [ ] Encriptação EBS ativada
- [ ] HTTPS/TLS configurado
- [ ] CloudTrail ativado
- [ ] VPC Flow Logs ativado
- [ ] GuardDuty ativado
- [ ] Backup regular (Snapshots/AMIs)
- [ ] Documentação de acesso

---

## 🔗 Próximos Passos

1. ➡️ **[Monitoramento](05-monitoramento.md)**
2. ➡️ **[Otimização de Custos](06-custos.md)**
3. ➡️ **[Troubleshooting](07-troubleshooting.md)**

---

**Última atualização**: Junho 2026  
**Autor**: João Iris  
**Licença**: MIT
