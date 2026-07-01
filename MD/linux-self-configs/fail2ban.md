# fail2ban

# **1. Preparação e verificação de telemetria**

Confirme que o SSH está enviando dados detalhados para o sistema de logs.


```bash
grep -i "^LogLevel" /etc/ssh/sshd_config
```

```text
LogLevel VERBOSE
```

Isso garante registro de:

- tentativas de login;
- chaves rejeitadas;
- usuários inexistentes;
- IPs de origem.

Se não estiver configurado neste modo, altere e execute:


```bash
sudo systemctl reload sshd
```

# **2. Configurar detector de intrusão**

O Fail2ban é uma ferramenta que lê os logs do sistema e, ao detectar um padrão de ataque, executa um comando de bloqueio no `iptables`.

# **a. Instalação fail2ban**

```bash
sudo apt update
sudo apt install -y fail2ban
```

Após a instalação, verifique o status do serviço e habilite-o para iniciar automaticamente junto com o sistema.

Garante que o Fail2ban inicie ao ligar o servidor e inicie o serviço agora:

```bash
sudo systemctl enable --now fail2ban`
```

Verifique status do sistema:

```bash
sudo systemctl status fail2ban
```

# **b. Blindagem base (jail.local)**

Crie um arquivo de sobreposição para personalizar as regras:

### **Atenção**

Nunca edite `jail.conf` diretamente.

```bash
sudo nano /etc/fail2ban/jail.local
```

**Aplique esta configuração otimizada:**

```text
[DEFAULT]
bantime  = 24h
findtime = 10m
maxretry = 2

backend = systemd
banaction = iptables-multiport

ignoreip = 127.0.0.1/8 ::1 # nunca bloquear a si mesmo

bantime.increment = true
bantime.factor = 2
bantime.maxtime = 1w

[sshd]
enabled = true
port    = ssh
logpath = %(sshd_log)s
```

- `bantime 1h` IP hostil fica fora por 1 hora
- `findtime 10m` Janela de tempo para contar as falhas (10 minutos)
- `maxretry 3` Quantidade de falhas permitidas antes do ban (3 falhas)
- `backend systemd` Lê direto do journald do sistema (mais confiável)
- `banaction` Cria uma cadeia (*chain)* específica no seu Firewall para os IPs banidos, mantendo suas regras originais limpas e organizadas.
- `ignoreip` Nunca banir a si mesmo ou a VPN.

# **c. Jail de SSH**

Ative explicitamente o jail de SSH:

```bash
sudo nano /etc/fail2ban/jail.local
```

# AO FINAL DO ARQUIVO, COLOQUE ESSE BLOCO [SSHD]

```text
[sshd]
enabled = true
port    = ssh
logpath = %(sshd_log)s`
```

Mesmo sem senha, este jail é essencial para:

- bloquear enumeração;
- reduzir ruído;
- proteger CPU contra floods.

Após salvar o arquivo, reinicie o serviço:

```bash
sudo systemctl restart fail2ban
```

# **3. Validação da resposta automática**

### **Ver status geral**

```bash
sudo fail2ban-client status
```
### **Ver status do jail SSH (quem está "preso" no SSH agora)**

```bash
sudo fail2ban-client status sshd
```

Você deve ver:

- IPs banidos (se houver);
- contadores ativos.

### Scripts

```bash
sudo nano f2b_status_log.sh
```

```text
sudo grep "Ban " /var/log/fail2ban.log
```

```bash
chmkd +x f2b_status_log.sh
```

```bash
sudo nano f2b_banstatus.sh
```

```text
sudo fail2ban-client status sshd
```

```bash
chmkd +x f2b_banstatus.sh
```


