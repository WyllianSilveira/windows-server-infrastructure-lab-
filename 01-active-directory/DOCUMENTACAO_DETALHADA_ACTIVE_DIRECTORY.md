# 🛠️ Implementação e Administração do Active Directory Domain Services (AD DS)

## 📌 Objetivo do Projeto
Demonstrar a capacidade prática de planejar, implantar e validar uma infraestrutura de identidade corporativa baseada no Windows Server 2019. Este projeto documenta a existência e a saúde do Controlador de Domínio (DC) ativo, mapeia a estrutura de objetos criados no diretório e valida o ciclo completo de autenticação e gerenciamento de ativos de rede.

---

## 🏗️ Topologia e Detalhes do Ambiente Virtual
*   **Hypervisor:** Oracle VM VirtualBox 
*   **Domain Controller (DC):** SERVIDOR1 (Windows Server 2019 Datacenter Evaluation)
*   **Client Machine:** Windows 10 Pro
*   **Nome do Domínio:** `empresa.local`
*   **Segmentação de Rede:** 
    *   `Ethernet`: IP Estático `192.168.100.10` (Rede interna do domínio / DNS Local)
    *   `Ethernet 2`: Endereço IPv4 atribuído por DHCP (Acesso externo / NAT)

---

## 🔍 Diagnóstico e Evidências do Ambiente

### 1. Configuração do Domínio e Domain Controller
**O que foi feito:** Promoção do servidor `SERVIDOR1` a Controlador de Domínio do domínio raiz `empresa.local`, com funções de Catálogo Global e DNS integradas no mesmo ativo.

*   **Comando de validação utilizado no PowerShell:**
    ```powershell
    Get-ADDomain | Select-Object Name, DomainMode, ForestMode
    Get-Service -Name NTDS, ADWS, Kdc | Select-Object Name, Status
    ```
*   **Evidência Visual:**
    ![Configuração do Domínio e Servidor Local](imagens/01-server-properties.png)
    *Nota: A imagem comprova as propriedades do sistema, o domínio ativo `empresa.local` e o endereçamento IP correspondente.*

---

### 2. Estrutura de Organizational Units (OUs)
**O que foi feito:** Organização do diretório através de Unidades Organizacionais (OUs) hierárquicas. O ambiente foi estruturado com sub-OUs dedicadas para segregar `Computadores` e `Usuários/Usuários_desativados` dentro de seus respectivos departamentos (`Adm`, `Financeiro`, `Ti` e `Vendas`). Essa abordagem segue as melhores práticas de mercado para garantir a aplicação granular e precisa de Objetos de Política de Grupo (GPOs) de acordo com o tipo de recurso.

*   **Comando de validação utilizado no PowerShell:**
    ```powershell
    Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName | Format-Table
    ```

    **Saída real do comando:**
    ```text
    Name                 DistinguishedName
    ----                 -----------------
    Domain Controllers   OU=Domain Controllers,DC=empresa,DC=local
    Vendas               OU=Vendas,DC=empresa,DC=local
    Computadores         OU=Computadores,OU=Vendas,DC=empresa,DC=local
    Usuários             OU=Usuários,OU=Vendas,DC=empresa,DC=local
    Adm                  OU=Adm,DC=empresa,DC=local
    Computadores         OU=Computadores,OU=Adm,DC=empresa,DC=local
    Usuários             OU=Usuários,OU=Adm,DC=empresa,DC=local
    Financeiro           OU=Financeiro,DC=empresa,DC=local
    Usuários             OU=Usuários,OU=Financeiro,DC=empresa,DC=local
    Computadores         OU=Computadores,OU=Financeiro,DC=empresa,DC=local
    Usuarios_desativados OU=Usuarios_desativados,DC=empresa,DC=local
    Ti                   OU=Ti,DC=empresa,DC=local
    Usuarios             OU=Usuarios,OU=Ti,DC=empresa,DC=local
    ```

*   **Evidência Visual:**
    ![Estrutura de OUs no Diretório](imagens/02-ou-structure.png)
    *Nota: O console 'Usuários e Computadores do Active Directory' comprova a árvore de OUs e sub-OUs hierárquicas personalizadas criadas no domínio empresa.local.*


### 3. Usuários, Grupos e Associações de Segurança
**O que foi feito:** Criação de contas de usuários para testes e grupos de segurança globais. Utilização do método de grupos para atribuição de acessos, garantindo eficiência na administração e aderência ao princípio do menor privilégio.

*   **Comando de validação utilizado no PowerShell:**
    ```powershell
    # Substitua "NomeDaSuaOU" por uma OU real do seu ambiente para listar os usuários
    Get-ADUser -Filter * -SearchBase "OU=NomeDaSuaOU,DC=empresa,DC=local" | Select-Object Name, SamAccountName
    ```
*   **Evidência Visual:**
    *(Insira o print das propriedades de um usuário do seu laboratório mostrando a aba 'Membro de')*
    ![Associação de Grupos e Usuários](imagens/03-users-groups.png)

---

### 4. Ingresso de Máquina Windows 10 no Domínio
**O que foi feito:** Configuração manual do adaptador de rede da máquina cliente (Windows 10) apontando o servidor DNS primário para o IP `192.168.100.10`, permitindo a resolução de nomes do AD e a execução do Join da estação no domínio `empresa.local`.

*   **Comando de validação utilizado no PowerShell do Servidor:**
    ```powershell
    Get-ADComputer -Filter * | Select-Object Name, OperatingSystem, Enabled
    ```
*   **Evidência Visual:**
    *(Insira o print da tela de propriedades do sistema do Windows 10 cliente mostrando o pertencimento ao domínio empresa.local)*
    ![Estação de Trabalho no Domínio](images/04-domain-join.png)

---

### 5. Validação de Login e Sessão do Usuário
**O que foi feito:** Realização do logon interativo no Windows 10 utilizando uma conta criada no Active Directory, confirmando que a estação está consultando o `SERVIDOR1` para autenticação Kerberos.

*   **Comando de validação utilizado no Prompt (CMD) da máquina cliente:**
    ```cmd
    whoami
    net user %username% /domain
    ```
*   **Evidência Visual:**
    *(Insira o print do CMD da máquina Windows 10 com o resultado do whoami mostrando 'empresa\nome-do-usuario')*
    ![Validação de Sessão do Usuário](images/05-user-login.png)

---

### 6. Administração Remota via RSAT (Remote Server Administration Tools)
**O que foi feito:** Configuração e uso das Ferramentas de Administração de Servidor Remoto (RSAT) instaladas no Windows 10 cliente, demonstrando a boa prática de gerenciar o Active Directory sem a necessidade de abrir sessões RDP ou logar localmente no Domain Controller.

*   **Como visualizar a evidência:** Abrir o console `dsa.msc` (Usuários e Computadores do AD) diretamente da sua estação de trabalho Windows 10 conectada ao domínio.
*   **Evidência Visual:**
    *(Insira o print do console de gerenciamento do AD rodando dentro da interface do Windows 10)*
    ![Administração via RSAT no Cliente](images/06-rsat-management.png)
