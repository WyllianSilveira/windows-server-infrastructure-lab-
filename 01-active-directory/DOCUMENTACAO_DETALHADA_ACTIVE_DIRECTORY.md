# 🛠️ Implementação e Administração do Active Directory Domain Services (AD DS)

## 📌 Objetivo do Projeto
Demonstrar a capacidade de planejar, implantar e administrar uma infraestrutura de identidade corporativa utilizando o Windows Server 2019. Este projeto documenta a configuração de um Controlador de Domínio (DC), a estruturação de Objetos de Diretório (OUs, Usuários e Grupos), o ingresso e validação de estações de trabalho clientes (Windows 10) e a administração descentralizada do ambiente.

---

## 🏗️ Topologia e Detalhes do Ambiente Virtual
*   **Hypervisor:** Oracle VM VirtualBox
*   **Domain Controller (DC):** Windows Server 2019 Standard
*   **Client Machine:** Windows 10 Pro
*   **Nome do Domínio:** `corp.local` *(Substitua pelo nome real do seu domínio)*
*   **Rede:** Rede Interna (Internal Network) / IPs Estáticos

---

## 🔍 Diagnóstico e Evidências do Ambiente

### 1. Configuração do Domínio e Domain Controller
**O que foi feito:** Validação da promoção do servidor a Controlador de Domínio, verificação das zonas de pesquisa e integridade dos serviços do catálogo global.

*   **Como verificar no seu ambiente:** No PowerShell do Windows Server, execute:
    ```powershell
    Get-ADDomain | Select-Object Name, DomainMode, ForestMode
    Get-Service -Name NTDS, ADWS, Kdc | Select-Object Name, Status
    ```
*   **Evidência Visual:**
    > *(Insira aqui o print do PowerShell com o resultado dos comandos acima ou uma captura da tela "Active Directory Users and Computers" mostrando o nó do seu domínio)*
    > `![Configuração do Domínio](images/01-domain-config.png)`

---

### 2. Estrutura de Organizational Units (OUs)
**O que foi feito:** Criação de uma estrutura hierárquica de Unidades Organizacionais segregando os recursos por departamentos (ex: Diretoria, TI, RH, Comercial) para facilitar a aplicação de políticas de grupo (GPOs) e delegação de controle.

*   **Como verificar no seu ambiente:** No PowerShell do Windows Server, execute para listar suas OUs personalizadas:
    ```powershell
    Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName | Format-Table
    ```
*   **Evidência Visual:**
    > *(Insira aqui o print da árvore de OUs expandida no console "Active Directory Users and Computers" ou o output do PowerShell)*
    > `![Estrutura de OUs](images/02-ou-structure.png)`

---

### 3. Usuários, Grupos e Associações
**O que foi feito:** Provisionamento de contas de usuários baseadas em cenários reais de negócios, criação de grupos de segurança globais e distribuição de privilégios via pertinência a grupos (Princípio do Menor Privilégio).

*   **Como verificar no seu ambiente:** Para listar os usuários de uma OU específica e os membros de um grupo:
    ```powershell
    # Listar usuários de uma OU (Substitua pelo nome da sua OU)
    Get-ADUser -Filter * -SearchBase "OU=TI,DC=corp,DC=local" | Select-Object Name, SamAccountName
    
    # Listar membros de um grupo específico
    Get-ADGroupMember -Identity "GG-TI-Admin" | Select-Object Name, samAccountName
    ```
*   **Evidência Visual:**
    > *(Insira aqui o print mostrando as propriedades de um usuário teste, a aba "Member Of" populada e a lista de usuários dentro do console ADUC)*
    > `![Usuários e Grupos](images/03-users-groups.png)`

---

### 4. Ingresso de Máquina Windows 10 no Domínio
**O que foi feito:** Configuração de rede da máquina cliente apontando o DNS para o Domain Controller, seguida pelo processo de Join da máquina Windows 10 Pro no domínio `corp.local`.

*   **Como verificar no seu ambiente:** No PowerShell do Windows Server, verifique se o computador consta na base do AD:
    ```powershell
    Get-ADComputer -Filter * | Select-Object Name, OperatingSystem, Enabled
    ```
*   **Evidência Visual:**
    > *(Insira aqui o print da tela de propriedades do Sistema no Windows 10 mostrando o domínio ativo, ou o computador registrado na pasta "Computers" do AD)*
    > `![Computador no Domínio](images/04-domain-join.png)`

---

### 5. Validação de Login e Sessão do Usuário
**O que foi feito:** Autenticação bem-sucedida na estação de trabalho utilizando as credenciais de um usuário comum do domínio, validando a comunicação de rede e o serviço de Kerberos/NTLM.

*   **Como verificar no seu ambiente:** Na máquina cliente Windows 10, abra o Prompt de Comando (CMD) e execute:
    ```cmd
    whoami
    net user %username% /domain
    ```
*   **Evidência Visual:**
    > *(Insira aqui o print do Windows 10 mostrando a tela de login com o formato DOMINIO\usuario ou o CMD executando o comando 'whoami')*
    > `![Login do Usuário](images/05-user-login.png)`

---

### 6. Administração Remota via RSAT (Remote Server Administration Tools)
**O que foi feito:** Demonstração de boas práticas de segurança ao administrar o Active Directory a partir do Windows 10 utilizando as ferramentas RSAT, evitando logins interativos via RDP diretamente no console do servidor.

*   **Como verificar no seu ambiente:** No Windows 10, certifique-se de que o recurso está instalado e abra o console do ADUC (`dsa.msc`) a partir do cliente.
*   **Evidência Visual:**
    > *(Insira aqui o print da tela inteira do seu Windows 10 mostrando o console "Active Directory Users and Computers" aberto e gerenciando o domínio corporativo)*
    > `![Administração via RSAT](images/06-rsat-management.png)`

