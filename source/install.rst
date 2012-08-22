.. _Swift: .. include:: etc/swift.conf


Instalação do Swift
===================

--------------------------
**Object Storage Nodes:**
--------------------------

Os object nodes foram instalados com o root filesystem sediado em uma LUN iSCSI, servida, para a cloud de LAB, pelo Filer de desenvolvimento (riofd06). Essa LUN contendo a instalação inicial do nó foi clonado em outras 6 LUNs, uma para cada "Object Storage". As LUNs foram clonadas e mapeadas como a seguir, para os seus respectivos hosts:

- riofd06:/vol/vol342/cittamp06lx06_root
- riofd06:/vol/vol342/cittamp06lx07_root
- riofd06:/vol/vol342/cittamp06lx08_root
- riofd06:/vol/vol342/cittamp06lx09_root
- riofd06:/vol/vol342/cittamp06lx10_root
- riofd06:/vol/vol342/cittamp06lx11_root

Nessa instalaçáo foi utilizado o "kickstart" (em riofb02a:/admfiler/2a/1/unix/tftpboot/pxelinux.cfg/centos6_64_SwiftWriter), para a instalação da LUN "golden", modelo para as demais. É basicamente uma instalação mínima com os pacotes e dependrências do Openstack Swift.

* Configurações Swift
   * object-server

.. line-block::

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

* Configurações específicas
