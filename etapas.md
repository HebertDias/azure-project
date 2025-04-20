# 📜 Etapas: Infraestrutura virtual no Azure

###### 🔺 _Esta é uma documentação utilizada para fins acadêmicos. Não é um tutorial e pode ser impreciso._

### 1. A primeira etapa consiste na criação de um Resource Group (RG), para agrupar todos os serviços que serão utilizados.

![RG](imagens/resourcegroup.png)

### 2. Com o RG criado, é necessária uma Virtual Network (VNET), que será responsável pela comunicação entre as máquinas virtuais (VMs), na faixa ‘192.168.105.0/24'.

![SUBNET](imagens/vm-ad01-ipnet.png)

###### 🔺 Os 4 primeiros IPs são reservados pelo Azure, logo, na minha faixa, minha primeira VM só poderá ser criada a partir do 192.168.105.4. Sendo assim, ficará da seguinte maneira:

| IP                  | Descrição             |
| ------------------- | --------------------- |
| 192.168.105.0 >     | Identificação da Vnet |
| 192.168.105.1 >     | Gateway               |
| 192.168.105.2 e 3 > | Serviços do Azure     |
| 192.168.105.255 >   | Broadcast             |

### 3. Com a VNET criada, criei minha primeira VM, onde a imagem será o Windows Server 2022 para ter acesso ao Server Manager, nas seguintes configurações:

![VM-AD](imagens/vm-ad01.png)

### 3.1 <b>Network Interface:</b> Direcionei à minha VNET e Subnet criada anteriormente. A conexão será por RDP.

![VM-AD-NETWORK](imagens/vm-ad01-network.png)

### 3.2. <b>IP Config:</b> O IP será estático, pois preciso que ele seja fixo, e terá o IP 192.168.105.4.

![VM-AD-IPCONFIG](imagens/vm-ad01-ipconfig.png)

### 3.3. Finalizando, baixei o arquivo RDP e loguei com a conta admin criada durante a criação da VM.

![VM-AD-RDP](imagens/vm-ad01-rdp.png)

## 🖥️ Configuração da VM01-AD

### 1. Ao acessar a VM, coletei o DNS Server pelo CMD. Neste caso, é o 168.63.129.16 (genérico do Azure).

![VM-AD-CMD](imagens/vm-ad01-cmd.png)

### 2. Com o DNS coletado, adicionei-o na NIC da minha VNET.

### ⚠️ O DNS do Azure será retirado logo após eu ter criado meu DNS. Sua inclusão aqui apenas configura boa prática.

![NIC-VNET](imagens/NIC.png)

### 2.1. Após um _ipconfig /renew_ e _ipconfig /all_, já é possível confirmar as inclusões feitas na NIC da VNET.

![VM-AD-CMD2](imagens/vm-ad01-cmd2.png)

## 🛠️ Server Manager

### 1. No Server Manager, acessei o _'Active Directory Domain Services'_ para instalar e configurar o AD DS, criando um domínio e DNS interno.

![SERVER-MANAGER](imagens/server-manager.png)

![SERVER-MANAGER2](imagens/server-manager2.png)

## 🖥️ VNET

### 1. Com meu DNS criado após a instalação do AD DS, como dito, apenas retirei o DNS genérico do Azure.

![VNET-NIC](imagens/vnet-nic.png)

## 🖥️ Configuração da VM02-CLIENT

### 1. Esta será a configuração da VM Client, do financeiro, que estará em nosso domínio e com acesso a uma GPO específica.

![VM-CLIENT](imagens/vm-client.png)

### 2. Após isso, bastou adicioná-la ao nosso domínio.

![VM-CLIENT-DOMAIN](imagens/vm-client-domain.png)

###### 🔺 A VM02 foi vinculada à VNET, herdando o DNS.

### 2.1 Importante ativar o Remote Desktop para Domain Users, para que usuários criados no AD possam logar nesta VM.

![VM-CLIENT-REMOTEDESKTOP](imagens/vm-client-remotedesktop.png)

## 🗂️ Active Directory

### 1. Com a VM02 inserida ao domínio, já é possível visualizá-la na OU "Computers".

![VM-CLIENT-COMPUTER](imagens/ad-computers.png)

### 2. Para manter um padrão de organização, criei uma OU para o setor da minha VM, e duas Sub-OUs, uma para as máquinas da área e outra para os usuários.

![VM-CLIENT-COMPUTER](imagens/ad-financeiro-user.png)
![VM-CLIENT-COMPUTER](imagens/ad-financeiro.png)

## 🛡️ GPOs

### 1. No Windows Server > Group Policy Management, criei uma GPO chamada 'sec.financeiro', vinculada à Sub-OU "user" da OU 'financeiro'.

![GPOs](imagens/gpos.png)
![GPOs](imagens/gpos2.png)

### 2. Em Scope, a GPO só poderá ser aplicada aos Domain Admins e ao próprio grupo 'sec.financeiro', permissões essas alteradas no Delegation.

![GPOs](imagens/gpo-scope.png)
![GPOs](imagens/gpo-delegation.png)

### 3. Antes de ir ao Drive Map para criar a GPO, a pasta 'Financeiro' foi compartilhada devidamente.

### ⚠️ Não configura boa prática — foi feito dessa maneira apenas para fins demonstrativos, considerando limitações do lab. O ideal seria utilizar um caminho dedicado a compartilhamentos, como `C:\Compartilhamentos\Financeiro`.

![GPOs](imagens/file-financeiro.png)

### 4. Com isso, a GPO foi criada com as especificações abaixo:

![GPOs](imagens/drivemap-creating.png)

### 5. No CMD da VM02 (com o usuário financeiro logado), ao dar um _'gpupdate /force'_, as políticas foram atualizadas e já é possível visualizar a GPO criada exclusiva para o setor Financeiro.

![GPOs](imagens/gpos3.png)
