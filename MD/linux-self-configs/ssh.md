# SSH

# **1. Preparação**

Conecte-se via SSH 

```bash
ssh -i ~/.ssh/minha_chave_especifica nome_usuario@srv01.seusite.com
```

### **2. Proteção do SSH**

# **a. Autenticação e identidade (Zero Trust)**

Edite:

```bash
sudo nano /etc/ssh/sshd_config
```

Apague ou comente as linhas existentes e aplique este padrão abaixo:

```text
# Força apenas chaves criptográficas; proíbe senhas e interatividade
AuthenticationMethods publickey
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
PermitEmptyPasswords no

# Proibir login remoto como root tanto usando senha ou via chave
PermitRootLogin no

# Restringe o acesso exclusivamente ao usuário informado
# AllowUsers <nome_usuario>

# Encerra sessões ociosas (5 min de inatividade)
ClientAliveInterval 300
ClientAliveCountMax 0

# Desabilita túneis e redirecionamentos desnecessários 
X11Forwarding no
AllowTcpForwarding no
AllowAgentForwarding no

# Logs detalhados para auditoria
LogLevel VERBOSE
```

Interpretação:

- `PasswordAuthentication no:` derruba brute force por senha.
- `PermitRootLogin no`: Isso impede que o usuário `root` faça login diretamente no servidor, inclusive com chave. Você poderá logar com o usuário criado anteriormente e usar o comando `sudo` para tarefas administrativas.
- `PubkeyAuthentication yes:` reforço explícito para permitir apenas autenticação por chave
- `PermitEmptyPasswords no`: Isso garante que contas com senhas vazias não possam se autenticar, adicionando outra camada de segurança.
- `ChallengeResponseAuthentication no`: Desabilita o método de autenticação de desafio-resposta, que pode ser suscetível a interceptações.
- `PasswordAuthentication no`: Desativa a autenticação de senha por SSH, exigindo que apenas chaves criptográficas sejam usadas para login.
- `AuthenticationMethods publickey`: Define que o único método de autenticação permitido é através de chave pública, garantindo que somente usuários com a chave correspondente possam acessar.
- `AllowUsers ops`: Isso permite que o usuário `ops` possa acessar o servidor. Você pode adicionar outros usuários à lista, separados por espaço.

# **b. Validar sintaxe e reiniciar SSH**

### **Importante**

Nunca reinicie o SSH sem validar antes e mantenha uma sessão SSH aberta enquanto testa a outra.

Verifique se houve erros de digitação:

```bash
sudo sshd -t
```

- Sem saída → configuração válida
- Com erro → corrija antes de continuar

Com as configurações salvas e validadas, reinicie o serviço:

```bash
sudo systemctl restart ssh
```
