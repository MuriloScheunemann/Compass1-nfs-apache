# 1º Trabalho - Estágio AWS & DevSecOps - Compass UOL
**Criar uma instância EC2 Linux com Apache e um compartilhamento NFS; Configurar um script de validação(Online/Offline) do Apache e enviar logs do status, de 5 em 5 minutos, para um diretório no compartilhamento NFS**

**Referências:** [Create an NFS server on Oracle Linux](https://docs.oracle.com/en/learn/create_nfs_linux/#introduction); [Network File System(RedHat Documentation)](https://access.redhat.com/documentation/pt-br/red_hat_enterprise_linux/6/html/storage_administration_guide/ch-nfs#s1-nfs-how); [Display Date And Time In Linux](https://www.cyberciti.biz/faq/linux-display-date-and-time/)
# Índice
- [Requisitos AWS](#requisitos-aws)
- [Requisitos Linux](#requisitos-linux)
- [Configuração do ambiente AWS (Via Console)](#configuração-do-ambiente-aws-via-console)
	+ [Gerando a chave de acesso](#gerando-a-chave-de-acesso)
	+ [Criando o Security Group](#criando-o-security-group)
	+ [Criando a instância EC2](#criando-a-instância-ec2)
	+ [Criando um IP fixo para a instância (Elastic IP)](#criando-um-ip-fixo-para-a-instância-elastic-ip)
- [Configuração do ambiente Linux](#configuração-do-ambiente-linux)
	+ [Configurando o NFS](#configurando-o-nfs)
	+ [Instalando o Apache](#instalando-o-apache)
	+ [Ajustando a data e hora do sistema](#ajustando-a-data-e-hora-do-sistema)
	+ [Criando o script](#criando-o-script)
	+ [Criando a rotina de execução do script com Cron](#criando-a-rotina-de-execução-do-script-com-cron)

## Requisitos AWS
* Gerar uma chave pública para acesso ao ambiente;
* Criar 1 instância EC2 com o sistema operacional Amazon Linux 2 (Família t3.small, 16 GB SSD);
* Gerar 1 elastic IP e anexar à instância EC2;
* Liberar as portas de comunicação para acesso público: (22/TCP, 111/TCP e UDP, 2049/TCP/UDP, 80/TCP, 443/TCP).
## Requisitos Linux
* Configurar o NFS entregue;
* Criar um diretorio dentro do filesystem do NFS com seu nome;
* Subir um apache no servidor - o apache deve estar online e rodando;
* Criar um script que valide se o serviço esta online e envie o resultado da validação para o seu diretorio no nfs;
	* O script deve conter - Data HORA + nome do serviço + Status + mensagem personalizada de ONLINE ou offline;
	* O script deve gerar 2 arquivos de saida: 1 para o serviço online e 1 para o serviço OFFLINE;
	* Preparar a execução automatizada do script a cada 5 minutos.
* Fazer o versionamento da atividade;
* Fazer a documentação explicando o processo de instalação do Linux.
## Configuração do ambiente AWS (via Console)
Pré-requisitos: uma conta AWS com AdministratorAccess ou, pelo menos, AmazonEC2FullAccess.  
### Gerando a chave de acesso
1. Dentro da console AWS, na seção de EC2, na aba *Network & Security*, selecionar *Key pairs*;
2. Clicar em *Create key pair*; depois, dar um nome à chave, selecionar o tipo *RSA*, o formato *.pem* e clicar em *create key pair*;
3. Salvar a chave num local seguro;
### Criando o Security-Group
1. Na aba *Network & Security*, selecionar *Security group*; clicar em *create security group*;
2. Dar um nome ao Security Group; selecionar a VPC default (ou uma VPC personalizada com internet gateway e tabela de roteamento);
3. Adicionar em *Inbound Rules* as seguintes regras de entrada:

|     Type      |   Protocol    |   Port Range   |   Source   | 
| ------------- | ------------- | -------------- | ---------- |
|  Custom TCP   |      TCP      |       22       |  0.0.0.0/0 |  
|  Custom TCP   |      TCP      |        111     |  0.0.0.0/0 |
|  Custom UDP   |      UDP      |        111     |  0.0.0.0/0 |
|  Custom TCP   |      TCP      |        2049    |  0.0.0.0/0 |
|  Custom UDP   |      UDP      |        2049    |  0.0.0.0/0 |
|  Custom TCP   |      TCP      |        80      |  0.0.0.0/0 |
|  Custom TCP   |      TCP      |        443     |  0.0.0.0/0 |
### Criando a instância EC2
1. Na seção de EC2, na aba *instances*, clicar em *instances*, e depois clicar em *launch instances* (no canto superior esquerdo);	
2. Selecionar a AMI Amazon Linux 2;
3. O tipo de instância T3.Small;
4. Selecionar a chave criada anteriormente;
5. Selecionar a VPC default (se necessário, criar uma nova VPC com internet gateway e tabelas de roteamento);
6. Selecionar o security group criado anteriormente;
7. Configurar um volume EBS do tipo gp3 SSD com 16GB;
8. Clicar em *Launch Instance*
### Criando um IP fixo para a instância (Elastic IP)
1. Na aba *Network & Security*, selecionar *Elastic IPs*; 
2. Clicar em *Allocate Elastic IP Address*; 

## Configuração do ambiente Linux
### Configurando o NFS
1. Verificar se o NFS está instalado com `systemctl status nfs-server.service`. Caso não estiver, executar `sudo yum install nfs-utils -y` e `sudo systemctl start nfs-server`; 
2. Criar um diretório de compartilhamento, dentro do diretório /mnt, com permissão total para todos. Executar: `sudo mkdir nfs -m 777`;
3. Dentro do diretório 'nfs', criar outro diretório, com permissão total para todos: executar `sudo mkdir murilo -m 777`;
4. Configurar compartilhamento NFS global do diretório 'nfs' editando /etc/exports. Adicionar a seguinte linha: `/nfs *(rw)`;
5. Restartar o serviço NFS para que as modificações sejam implementadas e habilitar inicialização no boot. Executar `systemctl restart nfs-server` e `systemctl enable nfs-server`;
### Instalando o Apache
1. Instalar o Apache. Executar: `sudo yum install httpd`;
2. Iniciar o Apache. Executar: `sudo systemctl start httpd`;
3. Habilitar a inicialização do Apache no boot. Executar: `sudo systemctl enable httpd`;
4. Verificar o status do Apache. Executar `systemctl status httpd`  
### Ajustando a data e hora do sistema
1. Verificar se a data e hora atuais do sistema estão corretas. Verificar com o comando `date`;
2. Se houver divergência, a região geográfica de referência deve ser alterada. Listar as regiões com `timedatectl list-timezones`;
3. Modificar a região com o comando `sudo timedatectl set-timezone nome_da_zona`. Para o caso de horário de Brasília: `sudo timedatectl set-timezone America/Sao_Paulo`.
### Criando o script
1. Criar um arquivo ShellScript no diretório raiz. Executar `sudo touch script.sh`;
2. Editar o arquivo do script, adicionando o seguinte código:
```
#!/bin/bash

DATAHORA=$(date +"%d/%m/%y-%T")
STATUS=$(systemctl is-active httpd)

if [ $STATUS = "active" ]; then
  echo -e "$DATAHORA - servico:Apache - status:Ativo - O Apache está\e[;32;01m ONLINE \e[m " >> /mnt/nfs/murilo/online
else
  echo -e "$DATAHORA - servico:Apache - status:Inativo - O Apache está\e[;31;01m OFFLINE \e[m " >> /mnt/nfs/murilo/offline
fi
```
3. Tornar o script executável. Executar `sudo chmod +x /script.sh`;
### Criando a rotina de execução do script com Cron
1. Abrir o editor do Cron. Executar `crontab -e`;
2. Criar uma rotina de execução do arquivo script.sh de 5 em 5 minutos. Adicionar a seguinte linha: `*/5 * * * * sudo bash /script.sh`
3. Salvar a tarefa criada e verificar com o comando `crontab -l`.

















