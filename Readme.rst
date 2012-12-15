MHA 
===

This Vagrant setup allows you to test easily MHA [#]_. MHA's wiki is already full of explanations on how to
use it.

However, this is a small summary.

Centos 6.3's Vagrant Box
------------------------

To get ready, the easiest to do is to download the Centos63 box I've used if you don't have any yet::

        wget http://dl.dropbox.com/u/11697684/centos63.box

And add it::

        vagrant box add centos63 centos63.box


Start the boxes
---------------

Because I didn't get time yet to find why grants are not added at the first provisioning, this is the
easiest way to start the boxes and have everything reading for MHA::

        vagrant up percona1
        vagrant provision percona1
        vagrant up percona2
        vagrant up percona3
        vagrant provision percona2
        vagrant provision percona3

Then you should have *percona1* as master and the other two as slaves (*percona2* and *percona3*)

MHA needs an already running replication setup.

Some MHA commands to test
-------------------------

Check [#]_ is the SSH communication between the machines is correct::

        masterha_check_ssh --conf=/etc/app1.cnf

Start MHA::

        masterha_manager --conf=/etc/app1.cnf

Then you can kill the master (*killall -9 mysqld_safe mysqld*) and see mha in action.

Check the status::

        masterha_check_status --conf=/etc/app1.cnf

example::

        [root@percona1 ~]# masterha_check_status --conf=/etc/app1.cnf
        app1 (pid:3969) is running(0:PING_OK), master:percona1

        OR

        [root@percona1 ~]# masterha_check_status --conf=/etc/app1.cnf
        app1 is stopped(2:NOT_RUNNING).


Stop MHA::

        masterha_stop --conf=/etc/app1.cnf


Switch master to a slave while the master is still alive::

        masterha_master_switch --master_state=alive --conf=/etc/app1.cnf --new_master_host=percona3

Switch master to a slave when the master is dead (mysql must not respond on it)::

        masterha_master_switch --master_state=dead --interactive=0 --wait_on_failover_error=0 --dead_master_host=percona3 --new_master_host=percona1



.. [#] http://code.google.com/p/mysql-master-ha/

.. [#] This needs to be run on percona1, it acts also as manager node
