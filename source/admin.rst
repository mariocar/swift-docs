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

*Adição de discos aos storage nodes*

Para adicionar-se mais discos ou aumentar a capacidade dos discos nos nodes de storage, é preciso recriar os arquivos de configuração e rebalancear-se o cluster, tal e qual feito na instalação. Como a adição/remoção de zones implica na redistribuição de partições pelo cluster, é aconselhável que esse procedimento seja feito paulatinamente, através do progressivo aumento do peso (weight) do novo/antigo dispositivo.

Ex. Adição de um disco de 3TB.




*Adição de nodes ao cluster*

*Remoção de discos dos storage nodes*

*Remoção de nodes do cluster*


Procedimentos de Recuperação de desastres:
------------------------------------------

blah blah blah
