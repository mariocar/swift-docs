.. _Swift: .. include:: etc/swift.conf
.. _XATTRS: http://docs.openstack.org/developer/swift/howto_installmultinode.html#configure-the-storage-nodes
.. _SwiftStorageDocs: http://docs.openstack.org/developer/swift/howto_installmultinode.html#configure-the-storage-nodes
.. _DOCL: http://docs.openstack.org/essex/openstack-compute/install/yum/content/ch_installing-openstack-object-storage.html
.. |OBJS| replace:: Swift Object Servers
.. |PROX| replace:: Swift Proxy Servers
.. |DOCL| replace:: Swift documentation

Administração do Serviço Swift
==============================

.. _monitoracao_swift:

Monitoração do serviço:
-----------------------

.. _procedimentos_de_mudanca:

Procedimentos de Mudança:
-------------------------

O Swift possui mecanismos de escalabilidade que permitem que sejam adicionados mais discos aos atuais servidores de objetos, ou mesmo mais servidores ao cluster atual. Esses procedimentos são bem simples, embora cautela seja recomendada durante sua condução para minimizar os impactos de performance que tais atividades impõem no cluster, a saber:

.. _adicao_de_discos:

Adição de discos aos storage nodes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Para adicionar-se mais discos ou aumentar a capacidade dos discos nos nodes de storage, é preciso recriar os arquivos de configuração e rebalancear-se o cluster, tal e qual feito na instalação. Como a adição/remoção de zones implica na redistribuição de partições pelo cluster, é aconselhável que esse procedimento seja feito paulatinamente, através do progressivo aumento do peso (weight) do novo/antigo dispositivo.

O procedimento em si é análogo ao de criação do cluster, descrito em : :ref:`criacao_dos_rings`

*Ex. Adição de um disco de 3TB. a uma zona* ::

     1. Adição de 25% dos 3000 = 750

        node-1$ mount /dev/sdc1 /srv/nodes/sdc1
	node-1$ swift-ring-builder account.builder add z1-node-1:6002/sdc1 750
	node-1$ swift-ring-builder container.builder add z1-node-1:6001/sdc1 750
	node-1$ swift-ring-builder object.builder add z1-node-1:6000/sdc1 750
	node-1$ swift-ring-builder account.builder rebalance
	node-1$ swift-ring-builder container.builder rebalance
	node-1$ swift-ring-builder object.builder rebalance

	node-1$ scp account.ring.gz node-2:/etc/swift/account.ring.gz
	node-1$ scp container.ring.gz node-2:/etc/swift/container.ring.gz
	node-1$ scp account.ring.gz node-2:/etc/swift/account.ring.gz
	node-1$ scp account.ring.gz node-3:/etc/swift/account.ring.gz
	node-1$ scp container.ring.gz node-3:/etc/swift/container.ring.gz
	node-1$ scp account.ring.gz node-3:/etc/swift/account.ring.gz
        ...
	node-1$ scp account.ring.gz node-N:/etc/swift/account.ring.gz
	node-1$ scp container.ring.gz node-N:/etc/swift/container.ring.gz
	node-1$ scp account.ring.gz node-N:/etc/swift/account.ring.gz

     2. Aguarde até que o I/O no cluster esteja estabilizado, aumente em 25% o peso do disco, e 
        repita o procedimento acima até que o peso seja 100%, ou seja, os 3000 para um HDD de 3GB

*Upgrade de discos do cluster*

O aumento da capacidade do cluster pela substituição de HDDs pequenos e/ou lentos por outros maiores/mais rápidos, também deve ser feita de forma gradual de modo a não gerar um alto assédio de I/O no cluster. Recomenda-se que o disco antigo seja removido pela gradual redução de seu peso no cluster, em passos de 25% (sugeridos) até que o mesmo chegue a 0. Após a remoção física do disco antigo e instalação do novo disco, o procedimento de adição gradual já descrito acima :ref:`adicao_de_discos`, deve ser observada.

*Adição de nodes ao cluster*



*Remoção de discos dos storage nodes*

*Remoção de nodes do cluster*


Procedimentos de Recuperação de desastres:
------------------------------------------

blah blah blah
