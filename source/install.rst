.. _Swift: .. include:: etc/swift.conf
.. _XATTRS: http://docs.openstack.org/developer/swift/howto_installmultinode.html#configure-the-storage-nodes
.. _DOCL: http://docs.openstack.org/essex/openstack-compute/install/yum/content/ch_installing-openstack-object-storage.html
.. |OBJS| replace:: Swift Object Servers
.. |PROX| replace:: Swift Proxy Servers
.. |DOCL| replace:: Swift documentation

Instalação do Swift
===================

O Swift foi instalado segundo a documentação em |DOCL|_

--------------------------
**Object Storage Nodes:**
--------------------------

Os object nodes foram instalados com o root filesystem sediado em uma LUN iSCSI, servida, para a cloud de LAB, pelo Filer de desenvolvimento (riofd06). Essa LUN contendo a instalação inicial do nó foi clonado em outras 6 LUNs, uma para cada "Object Storage". As LUNs foram clonadas e mapeadas como a seguir, para os seus respectivos hosts:

LUN GOLDEN
----------
- riofd06:/vol/vol342/swift_obj_root (LUN Golden)

LUNs CLONE - ROOTFS
-------------------
- riofd06:/vol/vol342/cittamp06lx06_root
- riofd06:/vol/vol342/cittamp06lx07_root
- riofd06:/vol/vol342/cittamp06lx08_root
- riofd06:/vol/vol342/cittamp06lx09_root
- riofd06:/vol/vol342/cittamp06lx10_root
- riofd06:/vol/vol342/cittamp06lx11_root

Nessa instalaçáo foi utilizado o "kickstart" (em riofb02a:/admfiler/2a/1/unix/tftpboot/pxelinux.cfg/centos6_64_SwiftWriter), para a instalação da LUN "golden", modelo para as demais. É basicamente uma instalação mínima de CentOS 6.3 x86_64, com os pacotes e dependências do Openstack Swift, na versão empacotada com a release Essex do Openstack (1.4.8).

Os dois HDDs locais disponíveis nos servidores, foram integralmente particionados e formatados com o FS XFS, pela necessidade de Extended Attributes (XATTRS_), conforme sugerido na documentação do Swift, e montados no mount point default de consumo do Swift, com os parâmetros recomendados na documentação, tanto para criação quanto para a montagem: ::

  Criação:
  mkfs.xfs -i size=1024 /dev/sda1
  mkfs.xfs -i size=1024 /dev/sdb1

  Montagem (/etc/fstab):
  /dev/sda1 /srv/node/sda1 xfs noatime,nodiratime,nobarrier,logbufs=8 0 0
  /dev/sdb1 /srv/node/sdb1 xfs noatime,nodiratime,nobarrier,logbufs=8 0 0



As interfaces de rede dos servidores foram configuradas como a seguir:

	*eth0* - Interface de acesso público (10.170.0.0/24 - DHCP)

	*eth1* - Interface de acesso privado - interconexão entre os |OBJS| e os |PROX| (192.168.33.0/24 - Estatica em função do IP na eth0)

-------------------
Configurações Swift
-------------------

.. compound::

   *object-server* ::

	[DEFAULT]
	bind_ip = 192.168.33.26
	workers = 24

	[pipeline:main]
	pipeline = object-server

	[app:object-server]
	use = egg:swift#object

	[object-replicator]

	[object-updater]

	[object-auditor]


*container-server* ::

	[DEFAULT]
	bind_ip = 192.168.33.26
	workers = 24

	[pipeline:main]
	pipeline = container-server

	[app:container-server]
	use = egg:swift#container

	[container-replicator]

	[container-updater]

	[container-auditor]

*account-server* ::

	[DEFAULT]
	bind_ip = 192.168.33.26
	workers = 24

	[pipeline:main]
	pipeline = account-server

	[app:account-server]
	use = egg:swift#account

	[account-replicator]

	[account-auditor]

	[account-reaper]

--------------
Tunnings do SO
--------------

	Visando privilegiar o throughput de I/O nos servidores na função de "object writers", alguns tunning foram feitos no ambiente a saber:

*Hardware*

Os servidores de objetos tiveram suas configurações de BIOS setadas para privilegiar I/O em detrimento de Acesso de memória::

  Advanced Options - Advanced Performance Tunning Options - QPI Bandwidth Optimization(RTID) - Optimized for I/O (32-16-40) \\ Balanced (32-24-32)
	
*Software*

.. compound::

     Os tunnings abaixo foram aplicados para o SO (sysctl): ::

	net.ipv4.ip_forward = 0
	net.ipv4.conf.default.rp_filter = 1
	net.ipv4.conf.default.accept_source_route = 0
	kernel.sysrq = 0
	kernel.core_uses_pid = 1
	net.ipv4.tcp_syncookies = 1
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
	net.ipv4.tcp_syncookies = 0
	net.ipv4.tcp_max_syn_backlog = 8192
	net.ipv4.conf.all.arp_ignore = 2
	net.ipv4.conf.all.arp_announce = 2
	net.ipv4.tcp_tw_recycle = 1
	net.ipv4.tcp_tw_reuse = 1
	net.ipv4.tcp_syncookies = 0
