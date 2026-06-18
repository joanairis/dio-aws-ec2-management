# 🔧 Troubleshooting: Resolvendo Problemas em EC2

Guia para diagnosticar e resolver problemas comuns com instâncias EC2.

---

## 🚫 Problema: Não Consegue Conectar via SSH

### Sintoma
```
ssh: connect to host 54.123.45.67 port 22: Connection timed out
```

### Causas e Soluções

#### 1. **Grupo de Segurança não permite SSH**

**Verificar**:
```
Console EC2 → Instância → Aba "Segurança"
→ Grupos de Segurança → Clique no grupo

Procure por regra SSH (22)
```

**Solução**:
```
1. Editar regras de entrada
2. Adicionar SSH (22)
3. Origem: Seu IP/32 (ex: 203.0.113.45/32)
4. Salvar
```

#### 2. **Arquivo .pem com permissão errada**

**Erro típico**:
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

**Solução**:
```bash
chmod 400 minha-chave.pem
# Permissões: Apenas leitura para dono
```

#### 3. **IP Público Errado**

**Verificar**:
```
1. Console EC2 → Instância
2. Procure por "IP público" (não IP privado)
3. Copie IP correto
```

**Conectar**:
```bash
ssh -i minha-chave.pem ubuntu@IP-PÚBLICO-CORRETO
```

#### 4. **Instância ainda iniciando**

**Verificar estado**:
```
Console EC2 → Instância → "Estado da instância"
Procure por: "running" ✅
```

**Se estiver "pending"**:
```
Aguarde 1-2 minutos para boot completo
Depois tente conectar novamente
```

---

## 🚫 Problema: Status Check Falhando

### Sintoma
```
Status Checks:
  System Status Check: ❌ Impaired
  Instance Status Check: ❌ Impaired
```

### Solução
```
1. Console EC2 → Instância
2. Clique direito → Parar instância
3. Aguarde ficar "stopped"
4. Clique direito → Iniciar instância
5. Aguarde "running"
6. Verificar status checks
```

---

## 📊 Comandos Úteis para Diagnóstico

```bash
# Sistema operacional
lsb_release -a
uname -a

# Hardware
lscpu          # Processador
free -h        # Memória
df -h          # Disco
lsblk          # Volumes

# Rede
ip addr        # IP
route -n       # Rotas
netstat -an    # Conexões

# Logs
dmesg          # Kernel logs
tail -f /var/log/syslog  # System logs
journalctl -xe # Journal logs

# Processos
ps aux
top
htop (instalar: sudo apt-get install htop)

# AWS Metadata
curl http://169.254.169.254/latest/meta-data/
```

---

## ✅ Checklist: Antes de Contactar Suporte

- [ ] Verificar status checks (System + Instance)
- [ ] Verificar Group de Segurança
- [ ] Verificar espaço em disco (df -h)
- [ ] Verificar memória (free -h)
- [ ] Ver logs (/var/log/syslog, dmesg)
- [ ] Tentar parar/iniciar instância
- [ ] Verificar metadados AWS
- [ ] Documentar problema com timestamps
- [ ] Coletar IDs (Instance ID, Volume ID)

---

**Última atualização**: Junho 2026  
**Autora**: Joana Iris  
**Licença**: MIT