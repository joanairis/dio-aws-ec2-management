# ⚙️ Gerenciar Instâncias EC2

Guia completo para gerenciar suas instâncias EC2 após criação.

---

## 🔗 Conectar à Instância

### Via SSH (Linux/Mac)

#### 1. Preparar a Chave SSH

```bash
# Mudar para o diretório onde a chave está
cd ~/Downloads

# Dar permissão de leitura (importante!)
chmod 400 minha-chave-ec2.pem
```

#### 2. Obter IP Público da Instância

1. Acesse o console EC2
2. Clique na instância
3. Anote o **"IP público"** (ex: 54.123.45.67)

#### 3. Conectar

```bash
# Sintaxe geral
ssh -i caminho/para/chave.pem usuario@ip-publico

# Exemplo com Ubuntu
ssh -i ~/Downloads/minha-chave-ec2.pem ubuntu@54.123.45.67

# Primeira conexão: Digite "yes" para aceitar a chave do host
```

**✅ Se conectou com sucesso**:
```
ubuntu@ip-172-31-xxx-xxx:~$
```

#### 4. Desconectar

```bash
exit
# ou
logout
```

### Via Windows RDP (Remote Desktop)

#### 1. Obter Senha

```
Console EC2 → Instância → Ações → Conectar → RDP
Clique em "Obter senha Windows"
Selecione arquivo .pem
Clique em "Descriptografar senha"
```

#### 2. Conectar

```
Windows:
1. Abra "Conexão de Área de Trabalho Remota"
2. Digite: [IP Público]:[Porta 3389]
3. Clique em "Conectar"
4. Usuário: Administrator
5. Senha: [Obtida no passo anterior]
```

### Via AWS Systems Manager Session Manager

**Vantagem**: Sem SSH/RDP, sem abrir portas

```bash
# Pré-requisito: IAM Role com permissão SSM

# Conectar
aws ssm start-session --target i-0123456789abcdef0

# Desconectar
exit
```

---

## 🎯 Operações Básicas

### 1. **Parar Instância** (Retenha dados)

```
Console EC2:
1. Clique direito na instância
2. Selecione "Parar instância"
3. Confirme

Via CLI:
aws ec2 stop-instances --instance-ids i-0123456789abcdef0 --region us-east-1
```

**Resultado**:
- ✅ Dados preservados
- ✅ Elastic IP permanece associado
- ⚠️ Armazenamento EBS ainda incorre custos (~$0.10/GB/mês)

**Quando usar**: Manutenção, para não perder dados

---

### 2. **Iniciar Instância** (Retomar)

```
Console EC2:
1. Clique direito na instância parada
2. Selecione "Iniciar instância"
3. Aguarde estado "running"

Via CLI:
aws ec2 start-instances --instance-ids i-0123456789abcdef0 --region us-east-1
```

**Nota**: IP público pode mudar (a menos que tenha Elastic IP)

---

### 3. **Reiniciar Instância**

```
Console EC2:
1. Clique direito na instância
2. Selecione "Reiniciar instância"
3. Aguarde ~1-2 minutos

Via CLI:
aws ec2 reboot-instances --instance-ids i-0123456789abcdef0 --region us-east-1
```

**Resultado**:
- ✅ IP público **NÃO** muda
- ✅ Dados preservados
- ✅ Como reiniciar um computador

---

### 4. **Terminar Instância** (Deletar)

```
Console EC2:
1. Clique direito na instância
2. Selecione "Terminar instância"
3. Confirme

Via CLI:
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0 --region us-east-1
```

**⚠️ AVISO**:
- ❌ **NÃO pode ser revertido!**
- ❌ Dados deletados permanentemente
- ✅ Custos param imediatamente
- ✅ IP público e Elastic IP liberados

---

## 🌐 Gerenciar IP Elástico (Elastic IP)

### Por que usar Elastic IP?

- ✅ IP público **fixo** (não muda ao parar/reiniciar)
- ✅ Pode ser transferido entre instâncias
- ✅ Essencial para DNS

### Alocar Elastic IP

```
Console EC2 → Endereços Elásticos → Alocar endereço

1. Domínio: VPC
2. Clique em "Alocar"
3. Você verá um novo IP (ex: 203.0.113.0)
```

### Associar a Uma Instância

```
1. Clique em "Associar endereço"
2. Selecione sua instância
3. Selecione a interface de rede (eth0)
4. Clique em "Associar"
```

### Desassociar

```
1. Clique no Elastic IP
2. Selecione "Desassociar endereço"
3. Confirme

⚠️ Sem instância associada, você paga ~$0.005/hora!
```

---

## 📊 Monitorar Instância

### Via Console AWS

```
1. Clique na instância
2. Abra aba "Monitoramento"
3. Veja gráficos de:
   - CPU Utilization
   - Network In/Out
   - Disk Read/Write
```

### Via CloudWatch

```
AWS Console → CloudWatch → Dashboards

Crie um dashboard com:
- CPU Utilization
- Network Traffic
- Disk Performance
- Status Checks
```

### Status Checks

```
1. Console EC2 → Instância → Status checks

Dois tipos de verificações:
- System Status: Hardware, rede
- Instance Status: SO, aplicação

Se falharem:
→ Parar e iniciar instância (novo hardware)
→ Contatar suporte AWS
```

---

## 🔧 Modificar Instância

### Mudar Tipo de Instância

```
Pré-requisito: Instância PARADA

1. Clique direito → Configurações → Alterar tipo de instância
2. Selecione novo tipo (ex: t3.small)
3. Clique "Aplicar"
4. Inicie a instância
```

⚠️ **Compatibilidade**:
- ✅ Pode aumentar (t3.micro → t3.small)
- ❌ Nem sempre pode diminuir (alguns tipos incompatíveis)
- ❌ Não pode mudar de AMD para Intel (ou vice-versa)

### Associar Elastic IP

Veja [seção acima](#gerenciar-ip-elástico-elastic-ip)

### Mudar Grupo de Segurança

```
1. Console EC2 → Instância
2. Aba "Segurança"
3. "Grupos de segurança" → "Editar grupos de segurança"
4. Selecione novo(s) grupo(s)
5. Salve
```

### Editar Tags

```
1. Console EC2 → Instância
2. Aba "Tags"
3. "Gerenciar tags"
4. Adicione/edite/delete tags
5. Salve
```

---

## 💾 Criar Backup

### Snapshots de EBS

```
Console EC2 → Volumes → Volume da sua instância
Clique direito → Criar snapshot

Nome: meu-servidor-backup-2026-06-18
Descrição: Backup completo do servidor web
```

**Depois**:
- Aguarde conclusão (pode levar minutos)
- Use para recuperação ou criar nova instância
- **Custa** ~$0.05 por GB armazenado

### Criar AMI (Imagem)

```
Console EC2 → Instância
Clique direito → Imagem e modelos → Criar imagem

Nome: meu-servidor-customizado
Descrição: Servidor web com Node.js instalado
Sem reinicialização: Marcar (opcional)
```

**Vantagem**:
- ✅ Salva configuração completa
- ✅ Rápido criar nova instância com mesmo setup
- ✅ Compartilhar com outros

---

## 🔐 Gerenciar Chaves SSH

### Listar Chaves

```bash
# Listar pares de chaves via CLI
aws ec2 describe-key-pairs --region us-east-1
```

### Importar Chave Existente

```
Console EC2 → Pares de chaves → Importar par de chaves

1. Nome: minha-chave-existente
2. Cole sua chave pública
3. Clique em "Importar par de chaves"
```

### Criar Novo Par

```
Console EC2 → Pares de chaves → Criar par de chaves

1. Nome: nova-chave
2. Tipo: RSA
3. Formato: .pem (Linux/Mac) ou .ppk (PuTTY Windows)
4. Clique em "Criar par de chaves"
5. Arquivo baixado automaticamente
6. **Guarde com segurança!**
```

---

## 📋 Operações em Lote

### Parar Múltiplas Instâncias

```bash
aws ec2 stop-instances \
  --instance-ids i-001 i-002 i-003 \
  --region us-east-1
```

### Terminar Múltiplas Instâncias

```bash
aws ec2 terminate-instances \
  --instance-ids i-001 i-002 i-003 \
  --region us-east-1
```

### Adicionar Tags em Lote

```bash
aws ec2 create-tags \
  --resources i-001 i-002 i-003 \
  --tags Key=Environment,Value=prod \
  --region us-east-1
```

---

## 🚨 Troubleshooting

### Problema: Não Consegue Conectar via SSH

**Causas Comuns**:

1. **Grupo de segurança não permite SSH**
   ```
   Solução: Editar grupo → Adicionar regra SSH (22) do seu IP
   ```

2. **Arquivo .pem com permissão errada**
   ```bash
   chmod 400 minha-chave.pem
   ```

3. **IP errado**
   ```bash
   # Copie o IP público correto do console
   ssh -i minha-chave.pem ubuntu@IP-CORRETO
   ```

4. **Instância ainda iniciando**
   ```
   Aguarde 1-2 minutos após o estado ficar "running"
   ```

### Problema: Instância não Inicia

**Causas Comuns**:

1. **Status Check falhando**
   → Parar e iniciar (novo hardware)

2. **Sem espaço em disco**
   → Aumentar volume EBS

3. **Erro de AMI**
   → Verificar logs de inicialização

---

## ✅ Checklist: Melhorias

- [ ] Adicionar Elastic IP
- [ ] Criar Snapshot/AMI regularmente
- [ ] Configurar CloudWatch alarmes
- [ ] Documentar configurações
- [ ] Testar procedimento de recuperação

---

## 🔗 Próximos Passos

1. ➡️ **[Práticas de Segurança](04-seguranca.md)**
2. ➡️ **[Monitoramento](05-monitoramento.md)**
3. ➡️ **[Otimização de Custos](06-custos.md)**

---

**Última atualização**: Junho 2026  
**Autor**: João Iris  
**Licença**: MIT
