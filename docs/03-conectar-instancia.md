# 🔗 Conectando à Instância EC2

## Requisitos

- ✅ Instância EC2 em estado "running"
- ✅ IP público atribuído
- ✅ Arquivo de chave privada (.pem)
- ✅ Grupo de segurança permite SSH (porta 22)

---

## Método 1: SSH via Terminal (Mac/Linux)

### Passo 1: Preparar a Chave

```bash
# Verificar se a chave existe
ls -la ~/.ssh/minha-primeira-chave.pem

# Dar permissão correta (IMPORTANTE!)
chmod 400 ~/.ssh/minha-primeira-chave.pem

# Verificar permissão
ls -la ~/.ssh/minha-primeira-chave.pem
# Esperado: -r-------- (400)
```

### Passo 2: Conectar

```bash
# Comando básico
ssh -i ~/.ssh/minha-primeira-chave.pem ec2-user@seu-ip-publico

# Exemplo real
ssh -i ~/.ssh/minha-primeira-chave.pem ec2-user@54.123.45.67

# Para Ubuntu (mude o usuário)
ssh -i ~/.ssh/minha-primeira-chave.pem ubuntu@seu-ip-publico
```

### Usuários Padrão por AMI:

```
Amazon Linux 2:    ec2-user
Ubuntu:            ubuntu
CentOS:            centos
Red Hat:           ec2-user
Windows:           Administrator (RDP, não SSH)
```

### Passo 3: Primeira Vez - Aceitar Fingerprint

```bash
# Você verá algo assim:
# The authenticity of host '54.123.45.67 (54.123.45.67)' can't be established.
# ECDSA key fingerprint is SHA256:abcd1234...

# Digite: yes
The authenticity of host '54.123.45.67 (54.123.45.67)' can't be established.
ECDSA key fingerprint is SHA256:abcd1234efgh5678ijkl9012mnop3456.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

### Resultado: Conectado!

```bash
ubuntu@ip-172-31-0-100:~$ 

# Agora você tem acesso total à instância
```

---

## Método 2: SSH via Windows (MobaXterm ou PuTTY)

### Opção A: MobaXterm (Recomendado)

1. **Baixar e Instalar**: https://mobaxterm.mobatek.net/
2. **Criar Nova Sessão**:
   - Clique em "Session" → "New session"
   - Escolha "SSH"
   - Host: seu-ip-publico
   - Username: ubuntu (ou ec2-user)
3. **Configurar Chave**:
   - Em "Advanced SSH settings"
   - Marque "Use private key"
   - Selecione seu arquivo .pem
4. **Conectar**

### Opção B: PuTTY (Tradicional)

1. **Converter Chave**:
   - Baixe PuTTYgen
   - Abra o arquivo .pem
   - "Save private key" → arquivo .ppk
   
2. **Configurar PuTTY**:
   - Host: seu-ip-publico
   - Port: 22
   - Em "SSH" → "Auth" → "Private key file": selecione .ppk
   - Salve a sessão
   
3. **Conectar**: Duplo clique na sessão salva

---

## Método 3: AWS Systems Manager Session Manager

### Alternativa Segura (sem pares de chaves expostos)

#### Pré-requisitos:

1. IAM Role na instância com permissão `AmazonSSMManagedInstanceCore`
2. Agente SSM instalado (vem pré-instalado em AMIs recentes)

#### Passos:

1. Vá para AWS Console
2. Busque "Systems Manager"
3. Clique em "Session Manager"
4. Selecione sua instância
5. Clique em "Start session"

#### Vantagens:

- ✅ Sem gerenciar pares de chaves
- ✅ Auditória integrada (CloudTrail)
- ✅ Sem abrir porta 22 para internet

---

## Método 4: EC2 Instance Connect (AWS Console)

### Conectar Diretamente do Console

#### Pré-requisitos:

- Ubuntu Server 6.2+
- Amazon Linux 2
- Grupo de segurança permite 0.0.0.0/0 porta 22

#### Passos:

1. Vá para EC2 → Instances
2. Selecione sua instância
3. Clique em "Connect"
4. Escolha aba "EC2 Instance Connect"
5. Clique em "Connect"
6. Terminal abrirá no navegador

---

## Operações Comuns na Instância

### Atualizar Sistema

```bash
# Para Ubuntu/Debian
sudo apt-get update
sudo apt-get upgrade -y

# Para Amazon Linux/CentOS
sudo yum update -y
```

### Instalar Software

```bash
# Exemplo: Instalar Node.js
sudo apt-get install -y nodejs npm

# Exemplo: Instalar Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### Verificar Informações da Instância

```bash
# Sistema operacional
cat /etc/os-release

# Informações de CPU
nproc
lscpu

# Memória
free -h

# Disco
df -h

# IP da instância
hostname -I

# Metadados AWS
curl http://169.254.169.254/latest/meta-data/
```

### Criar Arquivo de Teste

```bash
# Criar arquivo
echo "Olá, minha instância EC2!" > teste.txt

# Visualizar
cat teste.txt

# Remover
rm teste.txt
```

### Executar Servidor Web Simples

```bash
# Instalar Apache
sudo apt-get install -y apache2

# Iniciar serviço
sudo systemctl start apache2

# Verificar status
sudo systemctl status apache2

# Acessar no navegador
# http://seu-ip-publico
```

---

## Troubleshooting de Conexão

### Erro: "Permission denied (publickey)"

```bash
# Solução 1: Verificar permissão da chave
chmod 400 ~/minha-chave.pem

# Solução 2: Verificar se arquivo existe
ls -la ~/minha-chave.pem

# Solução 3: Usar opções verbose para debug
ssh -vvv -i ~/minha-chave.pem ubuntu@seu-ip
```

### Erro: "Connection timed out"

```
Causas possíveis:
1. IP público não está associado
   → Aloque Elastic IP
   
2. Grupo de segurança não permite SSH
   → Adicione regra: SSH (22) de seu IP
   
3. Instância ainda não está totalmente iniciada
   → Aguarde 2-3 minutos após "running"
   
4. IP correto não está sendo usado
   → Copie exatamente do console AWS
```

### Erro: "No such file or directory"

```bash
# Verifique caminho correto
ls -la ~/.ssh/

# Use caminho completo
ssh -i ~/.ssh/minha-chave.pem ubuntu@seu-ip

# Ou use caminho relativo
ssh -i ./minha-chave.pem ubuntu@seu-ip
```

### Não consegue acessar por SSH na primeira vez?

```bash
# Espere alguns minutos (instância iniciando)
sleep 120

# Verifique status dos checks
# AWS Console → Instância → Status checks
# Ambos devem estar "2/2 passed"

# Tente novamente
ssh -i ~/.ssh/minha-chave.pem ubuntu@seu-ip
```

---

## Desconectar da Instância

```bash
# Digite exit
exit

# Ou use Ctrl+D
^D
```

---

## Dicas de Segurança

### 1. Nunca Compartilhe a Chave Privada

```bash
# ❌ NÃO FAÇA ISTO
# Enviar por email
# Compartilhar no repositório Git (mesmo privado)
# Compartilhar em chat

# ✅ FAÇA ISTO
# Guarde localmente em ~/.ssh/
# Permissão 400 (somente leitura para você)
# Backup seguro (criptografado)
```

### 2. Use Elastic IP para Aplicações em Produção

```
IP público normal:
  - Muda quando instância é parada
  - Grátis enquanto instância está rodando
  
Elastic IP:
  - Não muda mesmo parando instância
  - Custa $0.005/hora se não estiver associado
  - Ideal para aplicações críticas
```

### 3. Restrinja SSH ao Seu IP

```
❌ Evitar: SSH de 0.0.0.0/0
✅ Usar: SSH do seu IP específico
   ou
✅ Usar: SSH de um bastion host
   ou
✅ Usar: Systems Manager (sem SSH exposto)
```

### 4. Atualize Regularmente

```bash
# Executar regularmente
sudo apt-get update
sudo apt-get upgrade -y

# Ou ative atualizações automáticas
sudo apt-get install -y unattended-upgrades
```

---

## Próximas Ações

Agora que está conectado, você pode:

1. **Instalar aplicações** (Node, Python, Docker, etc.)
2. **Configurar firewall** adicional
3. **Criar snapshots** para backup
4. **Monitorar performance** com CloudWatch
5. **Implementar auto-scaling**

---

**Próximo:** [Operações de Gerenciamento](04-gerenciar-instancia.md)
