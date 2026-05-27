# 🚀 Criando uma Instância EC2 - Passo a Passo

## Pré-requisitos

1. ✅ Conta AWS ativa (com acesso ao free tier se possível)
2. ✅ Permissões para criar instâncias EC2
3. ✅ Conhecimento básico de VPC e Security Groups

## Passo 1: Acessar o Console AWS

### Como Fazer:

1. Abra seu navegador e vá para [AWS Console](https://console.aws.amazon.com)
2. Faça login com sua conta AWS
3. Na barra de busca, digite **"EC2"**
4. Clique no serviço **EC2**

## Passo 2: Navegação até Instâncias

### Na Dashboard do EC2:

1. No menu lateral esquerdo, clique em **"Instances"** (Instâncias)
2. Você verá a lista de instâncias existentes
3. Clique no botão laranja **"Launch instances"** ou **"Executar instâncias"**

## Passo 3: Escolher uma AMI

### Imagem de Máquina (AMI)

Você verá várias opções pré-selecionadas:

```
Recomendações para Iniciantes:
  - Ubuntu Server 22.04 LTS (Free Tier eligible)
  - Amazon Linux 2 (Free Tier eligible)
```

### Passos:

1. Procure por **"Ubuntu Server 22.04 LTS"**
2. Verifique se tem o badge **"Free tier eligible"** (verde)
3. Clique em **Select** ou **"Selecionar"**

### Características da AMI Escolhida:

```yaml
Nome: Ubuntu Server 22.04 LTS
Arquitetura: 64-bit (x86)
Tipo de ROOT: EBS-backed
Acesso: SSH (porta 22)
Sistema: Debian-based, muito documentado
```

## Passo 4: Escolher Tipo de Instância

### Tipos Disponíveis (Free Tier):

```
┌─────────────────────┬──────────┬────────────┬─────────────┐
│ Tipo de Instância   │ vCPU     │ Memória    │ Armazenagem │
├─────────────────────┼──────────┼────────────┼─────────────┤
│ t2.micro            │ 1        │ 1 GB       │ EBS         │
│ t2.small            │ 1        │ 2 GB       │ EBS         │
│ t2.medium           │ 2        │ 4 GB       │ EBS         │
└─────────────────────┴──────────┴────────────┴─────────────┘
```

### Recomendação para Iniciantes:

- ✅ **t2.micro** (Free Tier)
- 1 vCPU com burst
- 1 GB de RAM
- Suficiente para testes e aprendizado

### Passos:

1. Na coluna esquerda, procure por **"t2.micro"**
2. Marque o checkbox ao lado
3. Clique em **"Next"** ou **"Próximo"**

## Passo 5: Configurar Detalhes da Instância

### Configurações Principais:

```yaml
Number of instances: 1

Network (VPC):
  - Selecione sua VPC padrão ou deixe default
  - Use a VPC padrão se for primeira vez

Subnet:
  - Deixe "No preference" ou selecione qualquer uma
  - A AWS selecionará automaticamente

Auto-assign Public IP:
  - IMPORTANTE: Clique em "Enable"
  - Caso contrário não conseguirá acessar via SSH

IAM instance profile:
  - Deixe vazio por enquanto
  - Pode adicionar depois se necessário

Monitoring:
  - Deixe desativado para reduzir custos
```

### Passos:

1. Configure as opções acima
2. Clique em **"Next"** ou **"Próximo"**

## Passo 6: Adicionar Armazenamento

### Configuração de EBS:

```yaml
Volume Type: gp3 (recomendado)
Size: 30 GB (Free Tier permite até 30 GB)
IOPS: 3000 (default é bom)
Throughput: 125 MB/s (default é bom)
Delete on Termination: ✅ (marque)
Encrypted: Deixe padrão
```

### Passos:

1. Verifique se o tipo é **gp3** ou **gp2**
2. Coloque **30 GB** de tamanho
3. Marque **"Delete on Termination"**
4. Clique em **"Next"** ou **"Próximo"**

## Passo 7: Adicionar Tags

### O que são Tags?

Tags são rótulos que ajudam a organizar seus recursos.

### Exemplo de Tags:

```yaml
Key: Name
Value: minha-primeira-instancia

Key: Environment
Value: learning

Key: Project
Value: DIO-EC2
```

### Passos:

1. Clique em **"Add tag"** ou **"Adicionar tag"**
2. Preencha os campos
3. Clique em **"Next"** ou **"Próximo"**

## Passo 8: Configurar Grupo de Segurança

### O que é Security Group?

É um firewall virtual que controla tráfego de entrada e saída.

### Configuração Recomendada:

```yaml
Security Group Name: minha-primeira-sg

Inbound Rules:
  - SSH (22): My IP (seu IP)
  - HTTP (80): Anywhere (0.0.0.0/0)
  - HTTPS (443): Anywhere (0.0.0.0/0)

Outbound Rules:
  - Deixe padrão (permite tudo)
```

### Passo a Passo:

1. Escolha **"Create a new security group"** ou **"Criar novo grupo"**
2. Dê um nome (ex: minha-primeira-sg)
3. Adicione regras:

#### Para SSH:

```
Type: SSH
Protocol: TCP
Port Range: 22
Source: My IP (pega seu IP automaticamente)
Description: SSH access
```

#### Para HTTP:

```
Type: HTTP
Protocol: TCP
Port Range: 80
Source: Anywhere - IPv4 (0.0.0.0/0)
Description: HTTP access
```

#### Para HTTPS (Opcional):

```
Type: HTTPS
Protocol: TCP
Port Range: 443
Source: Anywhere - IPv4 (0.0.0.0/0)
Description: HTTPS access
```

4. Clique em **"Next"** ou **"Próximo"**

## Passo 9: Revisar e Executar

### Checklist Final:

- ✅ AMI: Ubuntu Server 22.04 LTS
- ✅ Tipo: t2.micro
- ✅ IP Público: Habilitado
- ✅ Armazenamento: 30 GB
- ✅ Security Group: Configurado
- ✅ Tags: Adicionadas

### Passos:

1. Revise todas as configurações
2. Clique em **"Launch"** ou **"Executar instâncias"**

## Passo 10: Gerenciar Chave de Acesso (IMPORTANTE!)

### O que Acontece:

Após clicar em "Launch", a AWS pedirá para criar ou selecionar um par de chaves.

### Criar Nova Chave:

1. Selecione **"Create a new key pair"** ou **"Criar novo par de chaves"**
2. Dê um nome: `minha-primeira-chave`
3. Tipo: **RSA**
4. Formato: **.pem** (para Mac/Linux) ou **.ppk** (para Windows PuTTY)
5. Clique em **"Download Key Pair"** ou **"Fazer Download do Par de Chaves"**

### ⚠️ IMPORTANTE:

- 🔐 Guarde a chave em local seguro!
- 📁 Coloque em uma pasta específica (ex: `~/.ssh/`)
- 🔒 Nunca compartilhe a chave privada
- 💾 Faça backup da chave

### Passos após Download:

```bash
# No seu computador
cd ~/.ssh/

# Dar permissão correta (importante!)
chmod 400 minha-primeira-chave.pem

# Verificar permissão
ls -la minha-primeira-chave.pem
# Deve aparecer: -r-------- 1 user user ...
```

## Passo 11: Executar Instância

### Após o Download da Chave:

1. Clique em **"Launch instances"** ou **"Executar instâncias"**
2. Você verá uma mensagem de sucesso
3. Clique em **"View instances"** ou **"Ver instâncias"**

## Verificar Instância Criada

### Na Tela de Instâncias:

```
Você verá sua instância com status "running"

Informações Importantes:
  - Instance ID: i-xxxxxxxxxx
  - Instance State: running (verde)
  - Public IPv4: xxx.xxx.xxx.xxx
  - Public IPv4 DNS: ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com
```

## Próximo Passo

Agora que a instância foi criada, você pode:

1. **Conectar via SSH** (próximo documento)
2. **Configurar software** na instância
3. **Testar aplicações**

---

## Troubleshooting

### Status Check Failed

```
Problema: Instância está em "pending" por muito tempo
Solução: 
  1. Aguarde mais alguns minutos
  2. Se persistir, reinicie a instância
  3. Se ainda não funcionar, encerre e crie nova
```

### Cannot Connect via SSH

```
Problema: "Permission denied"
Solução:
  1. Verifique permissão do arquivo .pem (chmod 400)
  2. Verifique se o IP correto está em Security Group
  3. Verifique se a instância tem IP público
  4. Aguarde ~2 minutos após iniciar
```

### Instância Não Recebe IP Público

```
Problema: Public IPv4 não aparece
Solução:
  1. Encerre a instância
  2. Aloque um Elastic IP
  3. Associe ao ID da instância
  4. Reinicie
```

---

**Próximo:** [Conectar e Usar a Instância](03-conectar-instancia.md)
