.. _Swift: .. include:: etc/swift.conf
.. _XATTRS: http://docs.openstack.org/developer/swift/howto_installmultinode.html#configure-the-storage-nodes
.. _SwiftStorageDocs: http://docs.openstack.org/developer/swift/howto_installmultinode.html#configure-the-storage-nodes
.. _DOCL: http://docs.openstack.org/essex/openstack-compute/install/yum/content/ch_installing-openstack-object-storage.html
.. |OBJS| replace:: Swift Object Servers
.. |PROX| replace:: Swift Proxy Servers
.. |DOCL| replace:: Swift documentation

Instalação do Swift
===================

O Swift é um sistema de armazenamento distribuido e altamente redundante, projetado para atender uma única Região (links de baixa latência entre os servidores do Swift).  O cluster do Swift conta com dois tipos de nó (nodes), os |OBJS| e os |PROX|. A funcao dos primeiros é o armazenamento dos bancos de dados de configuração do Swift, metadados, contas de usuário e objetos propriamente ditos, enquanto os nós do segundo tipo prestam para o encaminhamento das requisições e serviço dos dados do cluster.


O Swift foi instalado segundo a documentação em |DOCL|_. Os endpoints resultantes dessa instalação são: ::

	https://swift.cumulus.dev.globoi.com/v1/AUTH_<tenant_id>/ 	(Interface de estáticos)
	http://swift.cumulus.dev.globoi.com:8080/v1/AUTH_<tenant_id>/   (Interface Swift)

As áreas de dados do Swift são designadas segundo os tenentes do Keystone. Cada "tenant" tem uma área discriminada pelo seu próprio UUID segundo a fórmula acima. Essas áreas podem ser divididas em "containers" (buckets, na terminologia AWS), e utilizadas para armazenamento de backups e conteúdo estático.

---------------------
Object Storage Nodes:
---------------------

Os Storage Nodes são os reais trabalhadores da infraestrutra do Swift. É neles que os dados de usuário e de configuração sáo armazenados e é neles que é garantida a redundância e disponibilidade dos dados.

ARMAZENAMENTO
-------------

Visando dedicar todo o storage local para uso pelo Swift, os object nodes foram instalados com o root filesystem sediado em uma LUN iSCSI, servida, para a cloud de LAB, pelo Filer de desenvolvimento (riofd06). Essa LUN contendo a instalação inicial do nó foi clonado em outras 6 LUNs, uma para cada "Object Storage". As LUNs foram clonadas e mapeadas como a seguir, para os seus respectivos hosts:

.. compound::

	LUN GOLDEN ::

	- riofd06:/vol/vol342/swift_obj_root (LUN Golden)

	LUNs CLONE - ROOTFS ::

	- riofd06:/vol/vol342/cittamp06lx06_root
	- riofd06:/vol/vol342/cittamp06lx07_root
	- riofd06:/vol/vol342/cittamp06lx08_root
	- riofd06:/vol/vol342/cittamp06lx09_root
	- riofd06:/vol/vol342/cittamp06lx10_root
	- riofd06:/vol/vol342/cittamp06lx11_root

As LUNs foram mapeadas para "igroups" (agrupamento de initiators iSCSI, no filer) contendo apenas o servidor "dono" da LUN e nomeado segundo o hostname do servidor: ::

    cittamp06lx06 (iSCSI) (ostype: linux):
        iqn.1994-05.com.redhat:cittamp06lx06 (logged in on: vif1)
    cittamp06lx07 (iSCSI) (ostype: linux):
        iqn.1994-05.com.redhat:cittamp06lx07 (logged in on: vif1)
    cittamp06lx08 (iSCSI) (ostype: linux):
        iqn.1994-05.com.redhat:cittamp06lx08 (logged in on: vif1)
    cittamp06lx09 (iSCSI) (ostype: linux):
        iqn.1994-05.com.redhat:cittamp06lx09 (logged in on: vif1)
    cittamp06lx10 (iSCSI) (ostype: linux):
        iqn.1994-05.com.redhat:cittamp06lx10 (logged in on: vif1)
    cittamp06lx11 (iSCSI) (ostype: linux):
        iqn.1994-05.com.redhat:cittamp06lx11 (logged in on: vif1)

Status final das LUNs após o mapeamento: ::

	LUN path                            Mapped to          LUN ID  Protocol
	-----------------------------------------------------------------------
	/vol/vol342/cittamp06lx06_root      cittamp06lx06           0     iSCSI
	/vol/vol342/cittamp06lx07_root      cittamp06lx07           0     iSCSI
	/vol/vol342/cittamp06lx08_root      cittamp06lx08           0     iSCSI
	/vol/vol342/cittamp06lx09_root      cittamp06lx09           0     iSCSI
	/vol/vol342/cittamp06lx10_root      cittamp06lx10           0     iSCSI

Para garantir o boot sem maiores configurações recomenda-se que a LUN de boot seja a primeira LUN apresentada aos servidores (LUN0), conforme acima.

Nessa instalaçáo foi utilizado o "kickstart" (em riofb02a:/admfiler/2a/1/unix/tftpboot/pxelinux.cfg/centos6_64_SwiftWriter), para a instalação da LUN "golden", modelo para as demais. É basicamente uma instalação mínima de CentOS 6.3 x86_64, com os pacotes e dependências do Openstack Swift, na versão empacotada com a release Essex do Openstack (1.4.8). Nessa LUN estão incluidas as configurações para os 5 iniciais nós do cluster, número mínimo de hosts recomendado para produção). Quando for necessário adicionar-se mais nós no cluster, pode-se clonar a LUN GOLDEN em uma nova LUN, criar um novo iGroup contendo o novo servidor, e mapear a LUN para esse recém criado igroup. Para informar o Swift sobre o novo nó, siga os procedimentos descritos em :ref:`procedimentos_de_mudanca`

Os dois HDDs locais disponíveis nos servidores, foram integralmente particionados e formatados com o FS XFS, pela necessidade de Extended Attributes (XATTRS_), conforme sugerido na documentação do Swift, e montados no mount point default de consumo do Swift, com os parâmetros recomendados na documentação, tanto para criação quanto para a montagem: ::

	  Criação:
	  mkfs.xfs -i size=1024 /dev/sda1
	  mkfs.xfs -i size=1024 /dev/sdb1

	  Montagem (/etc/fstab):
	  /dev/sda1 /srv/node/sda1 xfs noatime,nodiratime,nobarrier,logbufs=8 0 0
	  /dev/sdb1 /srv/node/sdb1 xfs noatime,nodiratime,nobarrier,logbufs=8 0 0


A disponibilidade dos dados é garantida por um esquema de réplicas, onde cada objeto é gravado em n-réplicas em zonas distintas, de modo a garantir a disponibilidade dos dados em caso de falha de até (n-replicas -1) zonas.

Cada servidor de objetos é uma "Zona" para o swift. As zonas são domínios de falha que devem ser configuradas do modo a ter-se tanta independência de recursos quanto possível (racks distintos, alimentação elétrica distinta, etc). O cluster deve ser disposto de modo a minimizar os efeitos de uma falha que afete mais de um nó do cluster ao mesmo tempo, sendo que a disponibilidade dos dados é garantida pelo número de réplicas configurado inicialmente,conforme já descrito.

As réplicas são feitas por intermédio do protocolo rSync. Cada servidor de objetos tem configurado em seu super-server (xinetd) um servidor de rsync que disponibliza os HDDs de cada zona para receber réplicas das demais zonas. O rsync foi configurado segundo sugerido na documentação do swift:


 */etc/xinetd.d/rsyncd.conf* ::

	# default: off
	# description: The rsync server is a good addition to an ftp server, as it \
	#       allows crc checksumming etc.
	service rsync
	{
		disable = no
		flags           = IPv4
		socket_type     = stream
		wait            = no
		user            = root
		server          = /usr/bin/rsync
		server_args     = --daemon
		log_on_failure  += USERID
	}

 */etc/rsyncd.conf* ::

	uid = swift
	gid = swift
	log file = /var/log/rsyncd.log
	pid file = /var/run/rsyncd.pid
	address = 192.168.33.xx

	[account]
	max connections = 2                 <- Valor sugerido pela documentação.
	path = /srv/node/
	read only = false
	lock file = /var/lock/account.lock

	[container]
	max connections = 2
	path = /srv/node/
	read only = false
	lock file = /var/lock/container.lock

	[object]
	max connections = 2
	path = /srv/node/
	read only = false
	lock file = /var/lock/object.lock

.. _criacao_dos_rings:

Criacao dos rings
"""""""""""""""""

Uma vez configurados os servidores (object, account e container), precisamos definir e informar ao Swift como particionar os discos, quantas réplicas fazer de cada objeto, etc. Essas configurações devem ser feitas com o utilitário "swift-ring-builder". ::

	$ cd /etc/swift
	$ swift-ring-builder account.builder   create <PARTITION_POWER> <REPLICAS> <MOVE_RESTRICTION_H>
	$ swift-ring-builder container.builder create <PARTITION_POWER> <REPLICAS> <MOVE_RESTRICTION_H>
	$ swift-ring-builder object.builder    create <PARTITION_POWER> <REPLICAS> <MOVE_RESTRICTION_H>

	$ swift-ring-builder object.builder    add z<ZONE>-<STORAGE_LOCAL_NET_IP>:6000/<DEVICE> <DISK_SIZE_GB>
	$ swift-ring-builder container.builder add z<ZONE>-<STORAGE_LOCAL_NET_IP>:6001/<DEVICE> <DISK_SIZE_GB>
	$ swift-ring-builder account.builder   add z<ZONE>-<STORAGE_LOCAL_NET_IP>:6002/<DEVICE> <DISK_SIZE_GB>

	$ swift-ring-builder account.builder

        PARTITION_POWER: 2^<PARTITION_POWER> = tamanho aproximado da partição.
	REPLICAS: número de réplicas que cada objeto terá no cluster.
        MOVE_RESTRICTION_H: número de horas a restringir que uma partição seja movida mais de uma vez.
        ZONE: número sequencial da zona.
        STORAGE_LOCAL_NET_IP: IP na rede de interconexão de baixa latência entres os nós.
        DEVICE: label que identifica o disco na árvore de montagem '/srv/node'.
        DISK_SIZE_GB: peso do disco, convencionado no número de GB do disco. 


REDE
----
As interfaces de rede dos servidores foram configuradas como a seguir:

 *eth0* - Interface de acesso público (10.170.0.0/24 - DHCP)
 *eth1* - Interface de acesso privado - interconexão entre os |OBJS| e os |PROX| (192.168.33.0/24 - Estatica em função do IP na eth0)



------------
Proxy Nodes:
------------
pacotes: openstack-swift-essex-proxy-essex-1.4.8-b3000, memcached-1.4.4-3.el6.x86_64

Descrição:
----------

Os proxy-nodes são os responsáveis por receber as requisições clientes do Swift. Pode-se ter tantos proxy-nodes quantos necessários em função da demanda, balanceados por um VIP. Todo tráfego é HTTP/HTTPS. 

Cacheamento automatico de estáticos:
------------------------------------

Para fins de testes, os proxy-nodes implementados no LAB Cumulus, são balanceados por um Varnish, com cacheamento default em 120 segundos para _todos_os_objetos_ servidos, indiscriminadamente, pela interface de estáticos. Essa configuração visa amortecer quaisquer picos de acesso via interface de estáticos. Os acessos internos do Swift, via porta 8080, são apenas balanceados e nunca cacheados (pipe).

Cacheamento de metadados:
-------------------------
Para fins de cacheamento de meta-dados para uso interno, o Swift usa instâncias de "memcache" em cada um de seus nós proxy. Cada proxy deve ser configurado para "enxergar" os memcaches dos demais nós de modo a criar uma rede redundante de processos memcached.


-------------------
Configurações Swift
-------------------

Cada cluster Swift deve ter um "Unique Identifier" (swift_hash_path_suffix), que o diferencie de outros clusters e que seja consistente entre os nós de cada cluster. Esse UUID deve ser armazenado no arquivo de configuração /etc/swift/swift.conf.

.. compound::

      */etc/swift/swift.conf:* ::

	[swift-hash]
	# random unique string that can never change (DO NOT LOSE)
	swift_hash_path_suffix =  d9fa0ad2ded1f0db


      */etc/swift/{object-server.conf|container-server.conf|account-server.conf}:* ::

	[DEFAULT]
	bind_ip = 192.168.33.26  <- Endereço privado de interconexão do cluster
	workers = 24             <- Número de threads = número de CPUs do host

	[pipeline:main]
	pipeline = object-server <- (ou container-server, ou account-server)

	[app:object-server]      <- (ou container-server, ou account-server)
	use = egg:swift#object   <- (ou swift#container, ou swift#account)

	[object-replicator]

	[object-updater]

	[object-auditor]

      */etc/swift/proxy-server.conf:* ::

	[DEFAULT]
	bind_port = 8080
	user = swift
	workers = 24 								<- Número de CPUs do servidor

	[pipeline:main]
	pipeline = catch_errors healthcheck cache swift3 s3token authtoken keystone staticweb proxy-server
	[app:proxy-server]
	use = egg:swift#proxy
	allow_account_management = true
	account_autocreate = true

	[filter:keystone]
	paste.filter_factory = keystone.middleware.swift_auth:filter_factory
	operator_roles = admin, swiftoperator

	[filter:cache]
	use = egg:swift#memcache
	memcache_servers = 10.170.0.31:11211 10.170.0.32:11211			<- Pool de memcacheds
	set log_name = cache

	[filter:catch_errors]
	use = egg:swift#catch_errors

	[filter:healthcheck]
	use = egg:swift#healthcheck

	[filter:authtoken]
	paste.filter_factory = keystone.middleware.auth_token:filter_factory
	delay_auth_decision = 1
	service_protocol = http
	service_port = 5000
	service_host = keystone.cumulus.dev.globoi.com				<- Identity Server (Keystone)
	auth_protocol = http
	auth_port = 35357
	auth_host = keystone.cumulus.dev.globoi.com
	admin_tenant_name = service						<- Tenant de serviços
	admin_user = swift                                     			<- Usuário do Swift no Keystone
	admin_password = 7a533b68-abd8-45a1-97c7-2feeb0e76871   		<- Senha do Usuário de serviço Swift

	[filter:staticweb]							<- Servidor de estáticos
	use = egg:swift#staticweb
	cache_timeout = 60
	set log_name = staticweb
	set log_facility = LOG_LOCAL0
	set log_level = INFO
	set access_log_name = staticweb
	set access_log_facility = LOG_LOCAL0
	set access_log_level = INFO

	[filter:swift3]								<- Emulador de S3
	use = egg:swift#swift3

	[filter:s3token]							<- Autenticação por tokens para S3
	paste.filter_factory = keystone.middleware.s3_token:filter_factory
	auth_port = 35357
	auth_host = keystone.cumulus.dev.globoi.com
	auth_protocol = http


      */etc/sysconfig/memcached:* ::

	PORT="11211"
	USER="memcached"
	MAXCONN="1024"
	CACHESIZE="4096"							<- Trade-off entre memória no servidor e acesso aos metadados.
	OPTIONS=""

--------------
Tunnings do SO
--------------

	Visando privilegiar o throughput de I/O nos servidores na função de "object writers", alguns tunning foram feitos no ambiente a saber:

*Hardware*

Os servidores de objetos tiveram suas configurações de BIOS setadas para privilegiar I/O em detrimento de Acesso de memória::

  Advanced Options - Advanced Performance Tunning Options - QPI Bandwidth Optimization(RTID) - Optimized for I/O (32-16-40) \\ Balanced (32-24-32)
	
*Software*

.. compound::

     Os tunnings abaixo foram aplicados para o SO (/etc/sysctl.conf): ::

	net.ipv4.ip_forward = 0
	net.ipv4.conf.default.rp_filter = 1
	net.ipv4.conf.default.accept_source_route = 0
	kernel.sysrq = 0
	kernel.core_uses_pid = 1
	kernel.msgmnb = 65536
	kernel.msgmax = 65536
	kernel.shmmax = 68719476736
	kernel.shmall = 4294967296
	net.ipv4.tcp_keepalive_time = 20
	net.ipv4.tcp_fin_timeout = 40
	net.ipv4.tcp_keepalive_intvl = 40
	net.ipv4.tcp_retries2 = 3
	net.ipv4.tcp_syn_retries = 2
	net.ipv4.ip_local_port_range = 1024 65000
	fs.file-max = 81920
	kernel.msgmni = 1024
	kernel.sem = 1000 32000 32 512
	kernel.shmmax = 2147483648
	net.ipv4.conf.all.arp_ignore = 2
	net.ipv4.conf.all.arp_announce = 2
	net.ipv4.tcp_tw_recycle = 1
	net.ipv4.tcp_tw_reuse = 1
	net.ipv4.tcp_syncookies = 0 			<- desligado nos Object Servers e ligado nos Proxy Servers
	net.ipv4.tcp_max_syn_backlog = 8192

.. _logging:

Logging:
--------

blah blah


