
Hardware Utilizado
==================
.. image:: _static/Swift_cumulus.png
   :scale: 75 %

# Object-writer nodes

  * 6x ProLiant BL460c G6  
     * 2x 10Gbps Nic
     * 2x 300GB 10KRpm SAS HDD
     * 24GB RAM
     * 24 CPUs Xeon(R) L5640 @ 2.27GHz

  * 6x 10GB iSCSI LUNs 
     * NetAPP Filer 

# Proxy nodes
  * 2x ProLiant BL460c G6
     * 2x 10Gbps Nic
     * 2x 300GB 10KRpm SAS HDD
     * 24GB RAM
     * 24 CPUs Xeon(R) L5640 @ 2.27GHz

.. table:: Truth table for "not"

   =====  =====
     A    not A
   =====  =====
   False  True
   True   False
   =====  =====

.. csv-table:: Frozen Delights!
   :header: "Treat", "Quantity", "Description"
   :widths: 15, 10, 30

   "Albatross", 2.99, "On a stick!"
   "Crunchy Frog", 1.49, "If we took the bones out, it wouldn't be
   crunchy, now would it?"
   "Gannet Ripple", 1.99, "On a stick!"


.. topic:: Topic Title

    Subsequent indented lines comprise
    the body of the topic, and are
    interpreted as body elements.

.. sidebar:: Sidebar Title
   :subtitle: Optional Sidebar Subtitle

   Subsequent indented lines comprise
   the body of the sidebar, and are
   interpreted as body elements.

.. note::
    
    This is a note somehow

.. warning::
    This is a warning

.. ATTENTION::
   Fudeu!

.. tip::
   Teste unknown tag

.. seealso::
   `Openstack Swift - Documentation <http://docs.openstack.org/essex/openstack-object-storage/admin/content/>`_
	OpenStack Object Storage Administration Manual

.. rubric:: Rubrica
   Paragraph Added git repo

.. code-block:: bash

   #!/bin/bash
   printf "Hello, World %s\n", "cedo"
   # this is a comment
   # This is a comment also

.. centered:: lICENSE AGREEMENT

.. hlist::
   :columns: 3

   * A list of
   * short items
   * that should be
   * displayed
   * Horrizontally

.. productionlist::
   try_stmt: try1_stmt | try2_stmt
   try1_stmt: "try" ":" `suite`
            : ("except" [`expression` ["," `target`]] ":" `suite`)+
            : ["else" ":" `suite`]
            : ["finally" ":" `suite`]
   try2_stmt: "try" ":" `suite`
            : "finally" ":" `suite`
.. compound::

   The 'rm' command is very dangerous.  If you are logged
   in as root and enter ::

       cd /
       rm -rf *

   you will erase the entire contents of your file system.

.. container:: custom

   This paragraph might be rendered in a custom way.



.. epigraph::

   No matter where you go, there you are.

   -- Buckaroo Banzai
