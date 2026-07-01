# Configuração da base do sistema
  
# **1. Preparação**

Conecte-se via SSH com a chave criada anteriormente:

```bash
ssh -i ~/.ssh/minha_chave_especifica root@ip_do_seu_servidor
```
# **2. Atualização inicial e pacotes base**

Instale ferramentas que auxiliam no diagnóstico e edição de arquivos no dia a dia.

```bash
sudo apt update && sudo apt -y upgrade && sudo apt -y install ca-certificates curl wget gnupg vim nano htop iotop jq unzip tar rsync dnsutils chrony fail2ban
```
- `ca-certificates, curl, gnupg`: base para repositórios e downloads confiáveis.
- `dnsutils / bind-utils`: necessários para depurar rotas, domínios e portas.
- `chrony`: tempo correto evita caos com TLS, logs e validação.
- `fail2ban`: camada mínima contra brute force.
- `vim` / `nano`: Editores de texto indispensáveis para configurar arquivos de sistema.
- `htop`: Monitor de processos interativo (CPU, RAM, Load Average).
- `iotop`: Monitor em tempo real de quem está consumindo a largura de banda do **Disco (I/O)**.
- `jq`: Processador de arquivos **JSON** via linha de comando (essencial para APIs e automações).
- `unzip` / `tar`: Ferramentas padrão para descompactar pacotes e backups.
- `rsync`: Protocolo eficiente para transferência e sincronização de arquivos entre servidores.
- `wget`: Utilitário para download de arquivos via HTTP/HTTPS/FTP.3. Identidade do servidor: hostname e timezone

# **3. Identidade do servidor: hostname e timezone**

### **Ajuste hostname**
```bash
sudo hostnamectl set-hostname <name>
```

Verifique a mudança executando o comando:

```bash
hostnamectl
```

### **Ajuste timezone (São Paulo BR, como exemplo)**

```bash
sudo timedatectl set-timezone America/Sao_Paulo
```

Verifique a mudança executando o comando:

```bash
timedatectl
```

### **Ajuste de locale (idioma e codificação)**

O Locale define como o sistema exibe datas, números e caracteres especiais.

**Recomendação**: Mantenha o sistema em Inglês (en_US.UTF-8).

```bash
localectl status
```

Defina para o padrão universal (Inglês Internacional com suporte a UTF-8):

```bash
sudo localectl set-locale LANG=en_US.UTF-8
```

*Nota: A mudança de locale só faz efeito completo após o próximo login ou reboot.*

# **4. Sincronismo de tempo (NTP)**

Aqui, usaremos o pacote `chrony`

### **Ative e valide o chrony**

```bash
sudo systemctl enable --now chrony 
```

```bash
sudo chronyc tracking
```

### **Se docker estiver instalado configure para executar 'docker' sem 'sudo'

1. Verifique se o grupo docker existe

```bash
getent group docker
```

Se não existir:

```bash
sudo groupadd docker
```

2. Adicione seu usuário ao grupo

```bash
sudo usermod -aG docker $USER
```

Execute:

```bash
newgrp docker
```

Isso abre um novo shell já com o grupo atualizado.

