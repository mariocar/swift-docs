.. _Swift: .. include:: etc/swift.conf
.. _XATTRS: http://docs.openstack.org/developer/swift/howto_installmultinode.html#configure-the-storage-nodes
.. |OBJS| replace:: Swift Object Servers
.. |PROX| replace:: Swift Proxy Servers


Instalação do Swift
===================

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

Os dois HDDs locais disponíveis nos servidores, foram integralmente particionados e formatados com o FS XFS, pela necessidade de Extended Attributes (XATTRS_), conforme sugerido na documentação do Swift, e montados no mount point default de consumo do Swift : 

  /srv/node/sd[a-z]1


As interfaces de rede dos servidores foram configuradas como a seguir:

*eth0* - Interface de acesso público (10.170.0.0/24)

*eth1* - Interface de acesso privado - interconexão entre os |OBJS| e os |PROX| (192.168.33.0/24)

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

o A BIOS foi configurada para maximizar o throughput de I/O
	
*
* Configurações específicas
