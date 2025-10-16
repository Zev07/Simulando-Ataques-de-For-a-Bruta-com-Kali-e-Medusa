![GitHub repo size](https://img.shields.io/github/repo-size/Zev07/Simulando-Ataques-de-For-a-Bruta-com-Kali-e-Medusa?style=for-the-badge)
![GitHub language count](https://img.shields.io/github/languages/count/Zev07/Simulando-Ataques-de-For-a-Bruta-com-Kali-e-Medusa?style=for-the-badge)
![GitHub last commit](https://img.shields.io/github/last-commit/Zev07/Simulando-Ataques-de-For-a-Bruta-com-Kali-e-Medusa?style=for-the-badge)
![GitHub license](https://img.shields.io/github/license/Zev07/Simulando-Ataques-de-For-a-Bruta-com-Kali-e-Medusa?style=for-the-badge)

<img src="images/diagrama-ambiente.png" alt="Diagrama do Ambiente de Laboratório">

### Sobre o projeto
> Projeto prático desenvolvido para o bootcamp de Cibersegurança da DIO. O objetivo é documentar a simulação de ataques de força bruta em um ambiente de laboratório controlado, utilizando Kali Linux, Medusa e Metasploitable 2 para identificar vulnerabilidades e propor medidas de segurança eficazes.

---

### 📖 Sumário
* [Tecnologias Utilizadas](#-tecnologias-utilizadas)
* [Ajustes e Melhorias](#-ajustes-e-melhorias)
* [Pré-requisitos](#-pré-requisitos)
* [Configurando o Ambiente](#-configurando-o-ambiente)
* [Executando os Cenários de Ataque](#-executando-os-cenários-de-ataque)
  * [Cenário 1: FTP Brute Force](#cenário-1-ftp-brute-force)
  * [Cenário 2: Web Form Brute Force (DVWA)](#cenário-2-web-form-brute-force-dvwa)
  * [Cenário 3: SMB Password Spraying](#cenário-3-smb-password-spraying)
* [Análise de Riscos e Mitigações](#-análise-de-riscos-e-mitigações)
* [Licença](#-licença)
* [Autor](#-autor)

---

### 🛠️ Tecnologias Utilizadas
- **Virtualização:** VirtualBox
- **Sistemas Operacionais:** Kali Linux (Atacante) e Metasploitable 2 (Vítima)
- **Ferramentas de Pentest:**
  - `Nmap` (Mapeamento de Rede)
  - `Medusa` (Ataque de Força Bruta)
  - `Enum4linux` (Enumeração de SMB)
- **Ambientes Vulneráveis:**
  - `vsftpd 2.3.4`
  - `Damn Vulnerable Web Application (DVWA)`
  - `Samba 3.0.20`

---

### 📊 Ajustes e Melhorias
O projeto foi concluído conforme o escopo do desafio.

- [x] Configuração do ambiente de laboratório virtual
- [x] Execução de ataque de força bruta em FTP
- [x] Execução de ataque de força bruta em formulário Web (DVWA)
- [x] Execução de password spraying em SMB

---

## 💻 Pré-requisitos

Antes de começar, verifique se você atendeu aos seguintes requisitos para replicar o ambiente:

- Você instalou a versão mais recente do **VirtualBox**.
- Você tem uma máquina **Linux / Windows / Mac** capaz de executar VMs.
- Você realizou o download das imagens do **Kali Linux** e **Metasploitable 2**.
- Você leu a documentação oficial do **[Medusa](http://foofus.net/goons/jmk/medusa/medusa.html)** para entender seus parâmetros.

---

## 🚀 Configurando o Ambiente
Para configurar o laboratório de pentest, siga estas etapas:

1. **Crie uma Rede Host-Only no VirtualBox:** Vá em `Arquivo > Gerenciador de Redes do Hospedeiro` e crie uma nova placa.
2. **Importe a VM do Kali Linux:** Configure a rede da VM para o modo "Host-Only Adapter".
3. **Crie e Configure a VM do Metasploitable 2:** Use o disco `.vmdk` existente e configure a rede também para "Host-Only Adapter".
4. **Valide a Conectividade:** Inicie ambas as VMs, obtenha o IP de cada uma (`ip a` ou `ifconfig`) e execute um `ping` do Kali para o Metasploitable para garantir que elas se comuniquem.

---

## ⚔️ Cenários de Ataque Executados

### 1. Ataque de Força Bruta em FTP (Porta 21)

O serviço `vsftpd 2.3.4` foi identificado como alvo. Um ataque de dicionário com Medusa foi realizado para encontrar credenciais válidas.

**Comando:**
```bash
medusa -h 192.168.56.101 -U usuarios.txt -P senhas.txt -M ftp
```

**Resultado:**
A credencial `msfadmin:msfadmin` foi comprometida com sucesso.

**Evidência:**
![Resultado do Ataque FTP](images/ataque-ftp-sucesso.png)

### 2. Ataque em Formulário Web (DVWA - Low Security)

O alvo foi o formulário de Brute Force do DVWA. O objetivo era descobrir a senha do usuário `admin`.

**Comando:**
```bash
medusa -h 192.168.56.101 -u admin -P senhas.txt -M http -m GET -m FORM:"/dvwa/vulnerabilities/brute/?username=^USER^&password=^PASS^&Login=Login" -m DENY-SIGNAL:"incorrect"
```

**Resultado:**
A senha `password` foi descoberta para o usuário `admin`.

**Evidência:**
![Resultado do Ataque DVWA](images/resultado-dvwa.png)

### 3. Password Spraying em SMB (Porta 445)

Primeiro, usuários foram enumerados com `enum4linux`. Em seguida, um ataque de *password spraying* foi realizado, testando um pequeno número de senhas comuns contra a lista de usuários encontrados.

**Comando:**
```bash
medusa -h 192.168.56.101 -U usuarios_smb.txt -P senhas_spray.txt -M smbnt
```

**Resultado:**
A credencial `msfadmin:msfadmin` foi novamente validada, mostrando que a mesma senha fraca estava sendo reutilizada em múltiplos serviços.

**Evidência:**
![Resultado do Ataque SMB](images/ataque-smb-sucesso.png)

---

## 🛡️ Recomendações de Mitigação

Com base nas vulnerabilidades exploradas, as seguintes contramedidas são recomendadas para fortalecer a segurança do ambiente:

1.  **Política de Senhas Fortes:** Implementar uma política que exija senhas com no mínimo 12 caracteres, incluindo maiúsculas, minúsculas, números e símbolos. Proibir o uso de nomes de usuário como senha.
2.  **Bloqueio de Contas (Account Lockout):** Configurar todos os serviços para bloquear temporariamente uma conta (ex: por 15 minutos) após 5 tentativas de login sem sucesso. Isso neutraliza a eficácia de ataques de força bruta.
3.  **Uso de Protocolos Seguros:** Substituir o FTP pelo **SFTP** ou **FTPS** para criptografar o tráfego de autenticação e dados.
4.  **Segurança em Aplicações Web:** Implementar **CAPTCHA** e **Autenticação de Múltiplos Fatores (MFA)** em formulários de login para impedir ataques automatizados.
5.  **Monitoramento e Alertas:** Configurar logs para registrar tentativas de login falhas e criar alertas para um número anômalo de falhas vindas de um mesmo IP, integrando ferramentas como o **Fail2Ban**.
