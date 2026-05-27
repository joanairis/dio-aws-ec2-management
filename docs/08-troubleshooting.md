# 🔧 Troubleshooting e Resolução de Problemas

## Problemas Comuns de Conectividade

### Erro: "Connection refused"

```
Causa: Porta não está aberta no security group

Solução:
1. Verifique Security Group
   Console AWS → EC2 → Security Groups
   
2. Adicione regra:
   Type: SSH
   Port: 22
   Source: Seu IP
   
3. Tente novamente
```

### Erro: "Connection timed out"

```
Causas possíveis:

1. Instância não tem IP público
   - Solução: Alocar Elastic IP
   
2. Security Group bloqueia porta
   - Solução: Adicionar regra de ingresso
   
3. Instância ainda iniciando
   - Solução: Aguardar 2-3 minutos
   
4. IP errado
   - Solução: Copiar IP correto do console
   
5. Network ACLs bloqueando
   - Solução: Verificar NACLs da subnet
```

### Erro: "Permission denied (publickey)"

```
Causas:

1. Permissão da chave errada
   chmod 400 ~/.ssh/minha-chave.pem
   
2. Chave errada para instância
   Verificar qual chave foi usada ao criar
   
3. Usuário errado
   Ubuntu: ubuntu
   Amazon Linux: ec2-user
   CentOS: centos
   
   Teste:
   ssh -i chave.pem ubuntu@seu-ip
   ssh -i chave.pem ec2-user@seu-ip
```

---

## Problemas de Performance

### CPU 100% Constantemente

```
Diagnóstico:
1. SSH na instância
2. top -b -n 1
3. Ver qual processo usa CPU

Solução:
1. Identificar processo (ps aux | grep [processo])
2. Otimizar código ou aumentar instância
3. Se bug: matar processo (kill -9 PID)
4. Reiniciar aplicação
```

### Disco Cheio (100%)

```
Diagnóstico:
1. df -h (ver espaço em disco)
2. du -sh /* (ver diretórios grandes)

Solução:
1. Limpar arquivos temporários
   sudo rm -rf /tmp/*
   sudo rm -rf /var/log/*.old
   
2. Comprimir logs antigos
   sudo gzip /var/log/syslog
   
3. Aumentar volume EBS
   - Parar instância
   - Criar snapshot do volume
   - Criar novo volume maior
   - Desanexar antigo, anexar novo
   - Expandir filesystem
```

### Memória Insuficiente

```
Diagnóstico:
1. free -h
2. top (ver processos usando memória)

Solução:
1. Curto prazo:
   - Matar processos desnecessários
   - Liberar cache (echo 3 | sudo tee /proc/sys/vm/drop_caches)
   
2. Médio prazo:
   - Otimizar aplicação (limitar conexões, cache)
   
3. Longo prazo:
   - Aumentar tipo de instância (mais RAM)
```

---

## Problemas de Sistema Operacional

### Instância Não Inicializa

```
Status Check: FAILED

Causas:
1. Kernel corrompido
2. Filesystem corrompido
3. Problema de hardware (raro)

Solução:
1. Parar e iniciar (tenta novo hardware)
2. Se não funcionar: reboot
3. Se ainda não: criar AMI e debugar
4. Último recurso: contatar AWS Support
```

### SSH Lento

```
Causa: DNS reverso lookup lento

Solução:
Editar /etc/ssh/sshd_config
Adicionar:
UseDNS no

Reiniciar SSH:
sudo systemctl restart ssh
```

### Sem Acesso à Internet

```
Diagnóstico:
1. Instância está em public ou private subnet?
2. curl ifconfig.me (testa internet)

Se private subnet:
- Precisa NAT Gateway/Instance
- Configure route table para 0.0.0.0/0 → NAT

Se public subnet:
- Verificar Internet Gateway
- Verificar route table
- Verificar Security Group outbound
```

---

## Problemas de Rede

### Conexão entre Instâncias Não Funciona

```
Causa provável: Security Group

Solução:
1. Source instância A
2. Destination instância B port (ex: 3306 para MySQL)
3. No Security Group de B, adicionar:
   Type: Custom TCP
   Port: 3306
   Source: Security Group de A
```

### Latência Alta

```
Diagnóstico:
1. ping google.com
2. traceroute google.com
3. mtr google.com

Causas:
1. Instâncias em AZs diferentes
   - Solução: mover para mesma AZ
   
2. Enhanced networking desativado
   - Solução: ativar se tipo suporta
   
3. Network interface saturada
   - Solução: aumentar tipo ou usar multiple ENIs
```

---

## Problemas de Segurança

### IP Bloqueado por "Too Many SSH Attempts"

```
Causa: Falhas de login repetidas

Solução:
1. Aguardar 1-2 horas
2. Ou resetar fail2ban no servidor
```

### Suspeita de Intrusão

```
1. ISOLAR:
   - Remover Elastic IP
   - Aplicar SG restritivo
   
2. INVESTIGAR:
   - Verificar CloudTrail
   - Verificar VPC Flow Logs
   - Verificar auth.log
   
3. REMEDIAR:
   - Criar AMI para análise forense
   - Encerrar instância
   - Criar nova limpa
   
4. COMUNICAR:
   - Contatar AWS Support
   - Se dados comprometidos: notificar usuários
```

---

## Problemas de Aplicação

### Aplicação Não Inicia

```
1. SSH na instância
2. Verificar logs:
   sudo systemctl status nome-app
   sudo journalctl -u nome-app -n 50
   
3. Verificar se porta está em uso:
   sudo netstat -tlnp | grep LISTEN
   
4. Verificar permissões:
   ls -la /caminho/app/
   
5. Verificar variáveis de ambiente:
   echo $VAR_NAME
```

### Aplicação Travada

```
1. Verificar processos:
   ps aux | grep nome-app
   
2. Se travado, matar:
   sudo kill -9 PID
   
3. Reiniciar:
   sudo systemctl restart nome-app
   
4. Verificar logs:
   tail -f /var/log/nome-app.log
```

---

## Comandos Úteis para Troubleshooting

### Informações do Sistema

```bash
# Informações de CPU
lscpu
nproc

# Informações de memória
free -h

# Informações de disco
df -h
du -sh *

# Processos
top -b -n 1
htop
ps aux

# Rede
netstat -tlnp
ss -tlnp
ipconfig -a
route -n

# Logs
dmesg
journalctl -n 50
tail -f /var/log/syslog
```

### Testes de Conectividade

```bash
# Ping
ping -c 4 google.com

# Traceroute
traceroute google.com

# DNS lookup
nslookup google.com
dig google.com

# Port scanning
nc -zv localhost 22
nmap -p 22 localhost

# Testar conectividade aplicação
curl http://localhost:8080
wget http://localhost:8080
telnet localhost 3306
```

---

## Quando Contatar AWS Support

```
✅ Contate AWS se:
- Instância foi hackeada (segurança)
- Hardware failure (System Status Check falha)
- Limite de conta atingido
- Erro de AWS (não do seu código)
- Performance issue sem causa óbvia

❌ Não contate AWS se:
- Erro no seu código
- Configuração do SO
- Problema de aplicação
- Falta de conhecimento geral
```

## Recursos de Suporte

```
1. AWS Documentation
   https://docs.aws.amazon.com/ec2/
   
2. AWS Forums
   https://forums.aws.amazon.com/
   
3. Stack Overflow
   Tag: amazon-ec2
   
4. AWS Support Center
   https://console.aws.amazon.com/support/
   
5. GitHub Issues
   Procure por repositórios relevantes
```

---

**Final:** Você agora tem conhecimento completo sobre EC2!