# Sred - Avaliação Final

## Introdução
Repositório destinado a avaliação final da matéria de SRED, onde foi requisitada fazê-la em três etapas. **Ao lado de cada etapa, estão os links para o passo a passo**.  

### 1º Etapa - [link](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/Readme.md)

a) Apresentar as tabelas de definições de nomes e IPs para todas as VMs

b) Criar o DNS Master e Slave, conforme o nome de domínio <grupoX>.<turma9yz>.ifala.local.
	
c) Criar a página do github do projeto do grupo.


### 2º Etapa:  [link](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/Readme.md)
a) Apresentar as tabelas de definições de nomes e IPs para todas as VMs incluindo as novas VMs para Web e Bd
	
b) Criar as entradas no DNS Master para as VMs Web e Bd
	
c) Configurar o servidor Web (LAMP)


### 3º Etapa:
a) Apresentar as tabelas de definições de nomes e IPs para todas as VMs incluindo as novas VMs para Web e Bd
	
b) implementar o GW: realizar a configuração do arquivo /etc/rc.local que reconfigura o firewall IPTABLES para redirecionar pacotes da rede interna para a rede externa.
	
c) realizar o port forwarding no rc.local para encaminhar dados para as portas 80 ( servidor Web), 445 e 139 (Samba) 
	
d) Reconfigurar as interfaces do SAMBA, WWW e BD para que passem somente pelo GW.
	
	
## 2. Definições Iniciais
	
 |          GRUPO 5					           
 |:------------------------------------|
 |         Nome dos integrantes        |
 |Aluno 1:   Maria Nathálya Veiga Rodrigues  |
 |Aluno 2:	 Diogo Bonfim Pinheiro	      |
 |Aluno 3:	 Sandrielly Amorim Silva      |


### CONFIGURAÇÕES DAS INTEFACES DE REDE		
|IP da Subrede:| 10.9.14.0/24  | 	192.168.14.32/29 |
|:-------------|:------------  | :---------------- |
|IP de Broadcast:|	10.9.14.255/24	|	192.168.14.39|		
|IP do GW:|	ens160 10.9.14.115	|ens192 	192.168.14.33|
|IP do SAMBA:	|ens160	10.9.14.104	| ens192	192.168.14.34|
|IP do NS1:	|ens160	10.9.14.121	| ens192	192.168.14.35|
|IP do NS2:	|ens160	10.9.14.129	| ens192	192.168.14.36|
|IP do WEB	|ens160	10.9.14.221	| ens192	192.168.14.37|
|IP do BD	|ens160	10.9.14.222	| ens192	192.168.14.38|
					

### Definição de Nomes e Domínio (<grupo>.<turma>.ifalara.local):		
|VM	Domínio | (zona): |grupo5.turma914.ifalara.local|
|:----------|:--------|:----------------------------|			
|Aluno29	|FQDN do GW:	|gw.grupo5.turma914.ifalara.local|
|Aluno04	|FQDN do SAMBA:	|smb.grupo5.turma914.ifalara.local|
|Aluno21	| FQDN do NS1:	| ns1.grupo5.turma914.ifalara.local|
|Aluno15 |	FQDN do NS2: | ns2.grupo5.turma914.ifalara.local|
|Grupo5vm1 | FQDN do WEB	| www.grupo5.turma914.ifalara.local |
|Grupo5vm2	| FQDN do BD	| bd.grupo5.turma914.ifalara.local |
	
## 3. Implementação dos Serivços de Rede

## 3.1 - Configuração da máquina ns1 
	
### Nela vamos configurar o serviço de resolução de nome (DNS) com o Bind9, utilizando-a como DNS Master
	
	
## 1º passo:
### Tem que mudar o nome da maquina para o nome escolhido, e que está na tabela: "ns1.grupo5.turma914.ifalara.local"
### Com o comando:
```
$ sudo hostnamectl set-hostname ns1.grupo5.turma914.ifalara.local
````
Agora vamos reiniciar a maquina para atualizar o nome:
````
$ sudo reboot
````

## 2º passo - Instalação do bind9.
### Antes de intalar o bind9, iremos fazer um update:
````
$ sudo apt update
````

### Agora vamos instalar o bind:

````
$ sudo apt-get install bind9 dnsutils bind9-doc
````
	
![1](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/1.png)
	
### Em seguida vamos verificar se o Bind9 está funcionando:
````
$ sudo systemctl status bind9
````
![2](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/2.png)

## 3º passo - Configuração dos arquivos de zonas.

### Antes deve-se criar o diretorio zones para alocar os nossos arquivos de zonas, com: 
````
$ sudo mkdir /etc/bind/zones
````
![3](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/3.png)

### Agora vamos copiar os arquivos para a pasta zones.

Zona Direta:
````
$ sudo cp /etc/bind/db.empty /etc/bind/zones/db.grupo5.turma914.ifalara.local
````
![4](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/4.png)

Zona Reversa:
````
$  sudo cp /etc/bind/db.127 /etc/bind/zones/db.10.9.14.rev
````
![5](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/5.png)

### Agora é que vamos editar o arquivo "db.grupo5.turma914.ifalara.local":
````
$ sudo nano /etc/bind/zones/db.grupo5.turma914.ifalara.local
````
![6](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/6.png)

### Nele vamos colocar os DNS e os IPs:

Configure, como está abaixo:
````
;
; BIND data file for internal network
;
$ORIGIN grupo5.turma914.ifalara.local.
$TTL	3h
@	IN	SOA	ns1.grupo5.turma914.ifalara.local. root.grupo4.turma914.ifalara.local. (
			      2022030900		; Serial
			      3h	; Refresh
			      1h	; Retry
			      1w	; Expire
			      1h )	; Negative Cache TTL
;nameservers
@	IN	NS	ns1.grupo5.turma914.ifalara.local.
@	IN	NS	ns2.grupo5.turma914.ifalara.local.

;hosts
ns1.grupo5.turma914.ifalara.local.	  IN	A	10.9.14.121
ns2.grupo5.turma914.ifalara.local.	  IN	A	10.9.14.129
smb.grupo5.turma914.ifalara.local.	  IN	A	10.9.14.104
gw.grupo5.turma914.ifalara.local.	  IN 	A	10.9.14.115
www.grupo5.turma914.ifalara.local.	  IN 	A	10.9.14.221
bd.grupo5.turma914.ifalara.local.	  IN 	A	10.9.14.222
````
### ANTES:
	
![7](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/7.png)

### DEPOIS:
	
![8](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/8.png)

### E vamos editar também o arquivo "db.10.9.14.rev":
````
$ sudo nano /etc/bind/zones/db.10.9.14.rev
````
![9](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/9.png)

### Nele vamos colocar os DNS e os IPs:


````
;
; BIND reverse data file of reverse zone for local area network 10.9.14.0/24
;
$TTL    604800
@       IN      SOA     grupo5.turma914.ifalara.local. root.grupo5.turma914.ifalara.local. (
                              2022030900         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

; name servers
@      IN      NS      ns1.grupo5.turma914.ifalara.local.
@      IN      NS      ns2.grupo5.turma914.ifalara.local.

; PTR Records
121   IN      PTR     ns1.grupo5.turma914.ifalara.local.
129   IN      PTR     ns2.grupo5.turma914.ifalara.local.
115   IN      PTR     gw.grupo5.turma914.ifalara.local.
104   IN      PTR     smb.grupo5.turma914.ifalara.local.
221   IN      PTR     www.grupo5.turma914.ifalara.local.
222   IN      PTR     bd.grupo5.turma914.ifalara.local.
````
### ANTES:
	
![10](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/10.png)

### DEPOIS:
	
![11](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/11.png)

## 4º passo - Ativação das zonas

### Para isso, devesse configurar o arquivo "named.conf.local":
````
$ sudo nano /etc/bind/named.conf.local
````
![12](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/12.png)

````
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

Nele vamos adicionas as seguintes linhas de codigos logo a baixo, pulando uma linha:

zone "grupo5.turma914.ifalara.local" {
	type master;
	file "/etc/bind/zones/db.grupo5.turma914.ifalara.local";
	allow-transfer{ 10.9.14.129; };  
	allow-query{any;};
};

zone "14.9.10.in-addr.arpa" IN {
	type master;
	file "/etc/bind/zones/db.10.9.14.rev";
	allow-transfer{ 10.9.14.129; };
};
````
Em zones tem que ter o nome de dominio escolhido. E em "allow-transfer", tem que colocar o IP é do ns2, ou seja Slave.

![13](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/13.png)


## 5º passo - Verificar a sintax dos arquivos que editamos.

### Para checar a sintax do arquivo named.conf.local usamos:
````
$ sudo named-checkconf
````

### Agora vamos entrar no diretório zones:
````
$ cd /etc/bind/zones
````
### Para verificar se o arquivo "db.grupo5.turma914.ifalara.local" está ok. Digite:
````
$ sudo named-checkzone grupo5.turma914.ifalara.local db.grupo5.turma914.ifalara.local
````
RESULTADO:
````
zone grupo5.turma914.ifalara.local/IN: loaded serial 1
OK
````
![14](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/14.png)

### Vamos verificar se o arquivo "db.10.9.14.rev" está ok também. Digite:
````
$ sudo named-checkzone 14.9.10.in-addr.arpa db.10.9.14.rev
````
RESULTADO:
````	
zone 14.9.10.in-addr.arpa/IN: loaded serial 1
OK
````
![15](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/15.png)


## 6º passo - Configuração para resolver apenas IPv4.

### Primeiro, temos que entrar no arquivo name e adicionaremos "-4" na linha "OPTINS=-u bind":
````
$ sudo nano /etc/default/named
````
![16](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/16.png)

Configure, como está abaixo:
````
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-4 -u bind"
````
![17](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/17.png)

## 7º passo - Reiniciar o bind, com:
````
$ sudo systemctl restart bind9
````
![18](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/18.png)

## 8º passo - Configuração das nossas interfaces, ens160 e ens192:
````
$ sudo nano /etc/netplan/00-installer-config.yaml
````
### Na interface ens160 devesse retirar os endereços de IPs e adiconar os indereçoes do Master e Sleve.

````
network:
  ethernets:
    ens160:
      addresses: [10.9.14.121/24]
      gateway4: 10.9.14.1
      dhcp4: false
      nameservers:
        addresses:
          - 10.9.14.121
          - 10.9.14.115
        search: [grupo5.turma914.ifalara.local]
    ens192:
      addresses: [192.168.14.35/29]
  version: 2
````
![19](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/19.png)

## 9º passo - Testar o DNS Marter.
````
$ dig @10.9.14.129 gw.grupo5.turma914.ifalara.local
````
OBS.: Na linha "ANSWER SECTION:" tem que mostra que foi resolvido, com o dominio e o IP.

![20](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/20.png)

### Para verificar se a interface 160 está funcionando, digite:
````
$ systemd-resolve --status ens160
````
### Para finalizar vamos ver se o nosso serviço DNS revolve o DNS do google:
````
$ ping google.com
````
![21](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns1/Galeria/21.png)


## 3.2 Na máquina ns2, é necessário configurar o Dns Slave com os dominios corretos

## 1º passo - Corrigir o nome da maquina, colocando de acordo com a planilha de acompanhamento
````
$ sudo hostnamectl set-hostname ns2.grupo5.turma914.ifalara.local
````
Agora vamos reiniciar a maquina para atualizar o nome:
````
$ sudo reboot
````
![1](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns2/Galeria/1.png)

## 2º passo - Instalação do bind9.
### Antes de intalar o bind9, iremos fazer um update:
````
$ sudo apt update
````

### Agora vamos instalar o bind:

````
$ sudo apt-get install bind9 dnsutils bind9-doc
````
![2](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns2/Galeria/2.png)

### Em seguida vamos verificar se o Bind9 está funcionando:

````
$ sudo systemctl status bind9
````

![3](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns2/Galeria/3.png)

## 3º passo - Configuração das interfaces de redes. Para isso digite:
````
$ sudo nano /etc/netplan/00-installer-config.yaml
````
### Mudar as configurações de acordo com o que está abaixo:
````
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens160:
      addresses: [10.9.14.115/24]
      gateway4: 10.9.14.1
      dhcp4: false
      nameservers:
        addresses:
          - 10.9.14.1		
          - 10.9.14.115
        search: [grupo5.turma914.ifalara.local]
    ens192:
      addresses: [192.168.14.36/29]
  version: 2
````
![4](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns2/Galeria/4.png)

### Para aplicar as mudanças, digite:
````
$ sudo netplan apply
````
### Verificar se está ok:
````
$ ifconfig
````
![5](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns2/Galeria/5.png)

## 4º passo - Verificar se o Bind está rodando
````
$ sudo systemctl status bind9
````
![6](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns2/Galeria/6.png)

### Devemos agora editar o arquivo "named.conf.local" informando que esse é o DNS Slave e qual o ip e o arquivo de DNS do Master. 

````
$ sudo nano /etc/bind/named.conf.local
````
![7](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns2/Galeria/7.png)

### Configure, como está abaixo:
````
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "grupo5.turma914.ifalara.local" {
        type slave;
        file "/etc/bind/zones/db.grupo5.turma914.ifalara.local";
        masters { 10.9.14.121; };
};

zone "14.9.10.in-addr.arpa" IN {
        type slave;
        file "/etc/bind/zones/db.10.9.14.rev";
        masters { 10.9.14.121; };
};
````
![8](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns2/Galeria/8.png)

### Para verificar o que foi escrito anteriormente, usasse o comando:
````
$ sudo named-checkconf
````
OBS.: Se não apareceu nem um erro então está ok.


## 5º passo - Testar para ver se está funcionando
````
$ systemd-resolve --status
````
![9](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns2/Galeria/9.png)

E testar também se o nosso DNS está resolvendo o nome do google.
````
$ ping google.com
````
![10](https://github.com/MNahVR/Sred-Final/blob/main/1Etapa/ns2/Galeria/10.png)
					

## 3.3 Nesta maquina "www" vamos configurar o servidor web usando a ferramenta Apache2 e configurar a maquina como cliente do serviço DNS. 

## 1º passo - Mudar o nome da máquina

  ```
  $ sudo hostnamectl set-hostname www.grupo5.turma914.ifalara.local
  ```

## 2º passo - Configurar máquina como cliente do serviço DNS.
### Para isso tem que configurar as interfaces de rede e adicionar os IPs de DNS e o campo "search".

  ```
  $ sudo nano /etc/netplan/00-installer-config.yaml
  ```
### ANTES:

![1](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/1.png)

### DEPOIS:

![2](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/2.png)

### Em seguida tem que aplicar as alterações.

  ```
  $ sudo netplan apply
  ```
![3](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/3.png)

## 3º passo - Agora vamos testar se está funcionando normalmente.

  ```
  $ dig @10.9.14.126 gw.grupo4.turma914.ifalara.local
  ```
  ![4](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/4.png)

  Na linha "ANSWER SECTION:" tem que aparecer que foi resolvido, com o dominio e o IP.

  ### Verificar se a interface 160 está funcionando:
  
  ```
  $ systemd-resolve --status ens160
  ```

  ### Ver se o serviço DNS consegue revolver o DNS do google:
  
  ```
  $ ping google.com
  ```
![5](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/5.png)

## 4º passo - Agora vomos instalar o apache e configurar para subir o servidor apacha.

  ### Primeiro, fazer update na máquina:
  
  ```
  $ sudo apt update
  ```
  ![6](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/6.png)
  ### Instalar o apache2:

  ```
  $ sudo apt install apache2
  ```
  ![7](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/7.png)

## 5º passo - Verificar se o servidor web está funcionando

  ```
  $ sudo systemctl status apache2
  ```
  ![8](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/8.png)

  ### Acessar o servidor web:
  Ir no navegador e colocar o IP da maquina "www"

  ### Caso não saiba seu IP basta digitar:
  
  ```
  $ hostname -i
  ```
  
  OBS.: Se o servidor web estiver rodando aparecerá, no navegador mostrará uma pagina do Apache2.
  
  ![9](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/9.png)

## 6º passo - criar o site.

  ### Primeiro tem que criar um diretório com o nome do nosso domínio, para isso usamos:
  
  ```
  $ sudo mkdir /var/www/grupo5.turma914.ifalara.local
  ```
  
  ### Atribuir a propriedade do diretório com a variável de ambiente $USER:
  
  ```
  $ sudo chown -R $USER:$USER /var/www/grupo.turma914.ifalara.local
  ```
  
  ### Configurar as permições dos webs hosts, com:
  
  ```
  $ sudo chmod -R 755 /var/www/grupo5.turma914.ifalara.local
  ```
  ![10](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/10.png)
  
  ### Para que o Apache apresente a nossa página vamos copiar um arquivo de configuração:
  
  ```
  $ sudo nano /etc/apache2/sites-available/www.grupo5.turma914.ifalara.local.conf
  ```
  ### Adicione as informações, como mostra a imagem:
  
  ![11](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/11.png)

  ### Habilitar o arquivo com a ferramenta a2ensite:
  
  ```
  $ sudo a2ensite www.grupo5.turma914.ifalara.local.conf
  ```
  ![12](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/12.png)

  ### Desabilitar o site padrão definido em 000-default.conf:
  
  ```
  $ sudo a2dissite 000-default.conf
  ```
  ![13](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/13.png)

  ### Procura de erros de configuração:
  
  ```
  $ sudo apache2ctl configtest
  ```
  ![14](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/14.png)

  ### Vamos modificar as permissões de dono da pasta "DocumentRoot" para "www-data", que é o usuário do apache.

  ```
  $ chown -R www-data:www-data www.grupo5.turma914.ifalara.local
  ```

  ### Acessar o navegador com o IP da maquina. 

  ### Criar página WEB  em HTML
  ````
  sudo nano /var/www/grupo5.turma914.ifalara.local/index.html
  ````
  ![19](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/19.png)

  ### Reiniciar o apache
  ````
  sudo systemctl restart apache2
  ````
  ![15](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/15.png)

  ### Digitar no navegador, o ip da maquina www, para ver a página html
  
  ![16](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/16.png)

## 7º passo - Instalação PHP

  ### Vamos fazer um update na maquina:

  ```
  $ sudo apt update -y
  ```

  ### Caso precise faça o upgrade na maquina:

  ```
  $ sudo apt upgrade -y
  ```
  ### Instale o PHP

  ```
  $ sudo apt install php7.4 libapache2-mod-php7.4 php7.4-mysql php-common php7.4-cli php7.4-common php7.4-common php7.4-json php7.4-opcache php7.4-readline
  ```
  ![17](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/17.png)

  ### Carregue o php
  ```
  $ sudo a2enmod php7.4
  ```
  ![18](https://github.com/MNahVR/Sred-Final/blob/main/2Etapa/www/Galeria/18.png)

  ### E reinicie o apache2

  ```
  $ sudo systemctl restart apache2
  ```

  Obs.: O php já está funcionando
