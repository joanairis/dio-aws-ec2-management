# 🚀 Criar uma Instância EC2

Um guia passo a passo completo para criar sua primeira instância EC2 na AWS.

---

## 📋 Pré-requisitos

- ✅ Conta AWS criada
- ✅ Login no AWS Management Console
- ✅ Free Tier ativo (opcional, mas recomendado)

---

## 🎯 Passo 1: Acessar o Console EC2

1. Faça login no [AWS Management Console](https://console.aws.amazon.com)
2. Na barra de busca, digite **"EC2"**
3. Clique em **"EC2"** nos resultados
4. No menu esquerdo, clique em **"Instâncias"**

---

## 🎯 Passo 2: Clicar em "Executar instâncias"

1. Clique no botão **"Executar instâncias"** (laranja)
2. Você será redirecionado para o assistente de criação

---

## 🎯 Passo 3: Selecionar AMI (Sistema Operacional)

### Opção Recomendada para Iniciantes

```
Nome: Ubuntu Server 22.04 LTS
Arquitetura: x86-64 (64-bit)
Elegível para Free Tier: ✅ Sim
```

**Como selecionar**:
1. Na seção "Escolher uma AMI"
2. Procure por "Ubuntu Server 22.04 LTS"
3. Clique no botão **"Selecionar"** ao lado dela

### Outras Opções

| AMI | Vantagens | Quando usar |
|---|---|---|
| **Ubuntu 22.04 LTS** | Popular, suporte LTS, comunidade | Recomendado para iniciantes |
| **Amazon Linux 2** | Otimizado para AWS | Quando precisa integração AWS |
| **Windows Server 2022** | Para aplicações Windows | Desenvolvimento Windows |
| **CentOS** | Compatível com RHEL | Ambiente corporativo |

---

## 🎯 Passo 4: Escolher Tipo de Instância

### Recomendação para Free Tier

```yaml
Tipo: t3.micro
CPU: 1 vCPU
Memória: 1 GB
Rede: Até 5 Gbps
Elegível para Free Tier: ✅ Sim (750 horas/mês)
```

**Como selecionar**:
1. Na seção "Tipo de instância"
2. Procure por **"t3.micro"** ou **"t2.micro"**
3. Selecione uma delas

### Comparação de Tipos Populares

| Tipo | vCPU | RAM | Custo/hora | Free Tier |
|---|---|---|---|---|
| **t2.micro** | 1 | 1 GB | ~$0.01 | ✅ Sim |
| **t3.micro** | 2 | 1 GB | ~$0.01 | ✅ Sim |
| **t3.small** | 2 | 2 GB | ~$0.02 | ❌ Não |
| **t3.medium** | 2 | 4 GB | ~$0.04 | ❌ Não |

---

## 🎯 Passo 5: Configurar Detalhes de Rede

### Configuração Básica

```yaml
Rede (VPC): VPC Padrão
Sub-rede: Padrão
IP Público: ✅ Ativado (Atribuir automaticamente)
IPv6: ❌ Desativado (opcional)
Placement Group: Nenhum (leave blank)
```

**Como fazer**:
1. Na seção "Configurar detalhes de instância"
2. Deixe as configurações padrão
3. Role para baixo

---

## 🎯 Passo 6: Adicionar Armazenamento

### Configuração Recomendada

```yaml
Tipo de Volume: gp3 (General Purpose 3)
Tamanho: 30 GB (elegível para free tier)
Encriptação: ❌ Não (por enquanto)
Delete on Termination: ✅ Sim (padrão)
```

**Como fazer**:
1. Na seção "Adicionar armazenamento"
2. Você verá um volume padrão (8 GB ou 30 GB)
3. Se for 8 GB, clique em "Editar" e mude para 30 GB
4. Deixe outras opções como padrão

**⚠️ Nota**: O free tier oferece até 30 GB de armazenamento EBS gp3 por mês

---

## 🎯 Passo 7: Adicionar Tags (Identificadores)

Tags ajudam a organizar e rastrear suas instâncias.

### Tags Recomendadas

```yaml
Chave: Name
Valor: minha-primeira-instancia

---

Chave: Environment
Valor: dev

---

Chave: Project
Valor: DIO-EC2-Management
```

**Como adicionar**:
1. Na seção "Adicionar tags"
2. Clique em **"Adicionar nova tag"**
3. Preencha "Chave" e "Valor"
4. Repita para cada tag

---

## 🎯 Passo 8: Configurar Grupo de Segurança

Um grupo de segurança funciona como um firewall.

### Opção 1: Usar Grupo Padrão (Simples)

```yaml
Atribuir grupo de segurança: Selecionar um existente
Grupo: default
```

⚠️ **Nota**: O grupo padrão é menos seguro. Use apenas para teste.

### Opção 2: Criar Novo Grupo (Recomendado)

```yaml
Atribuir grupo de segurança: Criar um novo

Nome: web-server-sg
Descrição: Grupo para servidor web

Regras de Entrada:
  - SSH (22): De [Seu IP]/32
  - HTTP (80): De 0.0.0.0/0
  - HTTPS (443): De 0.0.0.0/0

Regras de Saída (padrão):
  - Todos os protocolos: Para 0.0.0.0/0
```

**Como criar regras**:
1. Na seção "Configurar grupo de segurança"
2. Selecione "Criar um novo"
3. Preencha nome e descrição
4. Para cada regra:
   - Tipo: Selecione (SSH, HTTP, etc.)
   - Protocolo: Automático
   - Intervalo de portas: Automático
   - Origem: Selecione (0.0.0.0/0 para público, seu IP para SSH)
5. Clique em "Adicionar regra" para mais

### ⚠️ Segurança: Como Descobrir Seu IP

```bash
# No terminal/prompt do seu computador:
curl https://checkip.amazonaws.com
# ou
wget -qO- https://checkip.amazonaws.com
```

---

## 🎯 Passo 9: Revisar e Executar

### Revisão Final

Verifique:
- ✅ AMI: Ubuntu Server 22.04 LTS
- ✅ Tipo: t3.micro ou t2.micro
- ✅ Armazenamento: 30 GB
- ✅ Grupo de Segurança: Configurado com SSH, HTTP, HTTPS
- ✅ Tags: Name, Environment, Project

### Selecionar Chave SSH

1. Na seção "Revisar instância"
2. Role até "Selecionar par de chaves existente"
3. **Opções**:
   - Se você tiver uma chave: Selecione
   - Se não tiver: Clique em "Criar novo par de chaves"

### Criar Novo Par de Chaves

Se não tiver uma chave SSH:

```
Nome do par: minha-chave-ec2
Tipo de chave: RSA
Formato: .pem (para Linux/Mac) ou .ppk (para Windows PuTTY)
```

**Ações após criar**:
1. Clique em **"Criar par de chaves"**
2. O arquivo `.pem` será **baixado automaticamente**
3. ⚠️ **Guarde este arquivo com segurança!**
4. Não compartilhe nem perca

### Executar

1. Clique em **"Executar instâncias"**
2. Aguarde 30-60 segundos
3. Você verá a mensagem de sucesso

---

## ✅ Sua Instância Foi Criada!

### Próximas Ações

1. Aguarde estado **"running"** (pode levar 30-60 segundos)
2. Clique no ID da instância para ver detalhes
3. Anote:
   - **IP Público**: Para conexão SSH
   - **IP Privado**: Para comunicação interna
   - **DNS Público**: Nome DNS

---

## 🔗 Próximos Passos

1. ➡️ **[Gerenciar Instâncias](03-gerenciar-instancia.md)**
2. ➡️ **[Práticas de Segurança](04-seguranca.md)**
3. ➡️ **[Monitoramento](05-monitoramento.md)**

---

## 💡 Dicas Úteis

### Calculadora de Custos
- Acesse: https://calculator.aws/
- Simule seus custos antes de criar

### Monitoramento Gratuito
- CloudWatch: Métricas gratuitas por 15 dias
- EC2 Console: Status checks gratuitos

### Limpeza após Teste
```
Quando terminar de testar:
1. Clique direito na instância
2. Selecione "Encerrar instância"
3. Confirme

⚠️ Isso deletará a instância e dados!
```

---

**Última atualização**: Junho 2026  
**Autora**: Joana Iris  
**Licença**: MIT