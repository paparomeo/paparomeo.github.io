.. title: Minimal Ubuntu Precise
.. slug: minimal-ubuntu-precise
.. date: 2012-08-24
.. tags: blog
.. author: Pedro Romano
.. link:
.. description:
.. category: linux, ubuntu

Minimal Ubuntu Precise
======================

The purpose of the minimal Ubuntu installation is to server as a base that can
be customised and configured automatically and reproducibly using a
`configuration management tool
<https://en.wikipedia.org/wiki/Comparison_of_open_source_configuration_management_software>`_
(in my case `Chef <https://www.opscode.com/chef>`_ in its `Solo
<http://wiki.opscode.com/display/chef/Chef+Solo>`_ incarnation). For virtual
machines, these manual steps can be easily automated with tools such as
`Vagrant <https://vagrantup.com/>`_ or `Ubuntu's JeOS VM Builder
<https://help.ubuntu.com/12.04/serverguide/jeos-and-vmbuilder.html>`_, but
since my virtual server plan only supports booting the VM from a CD as the
deployment method, I chose the minimal Ubuntu installation as the simplest
alternative for the base installation.

These instructions are based on the installation of the `Ubuntu 12.04 LTS
(Precise Pangolin) Minimal CD
<https://help.ubuntu.com/community/Installation/MinimalCD/>`_ and assume that
the installed Ubuntu system will be the only one on the machine. After booting
from the installation medium with the minimal CD image, the following steps
were executed:

#. On the **Installer boot menu** choose: ``Install``
#. On the **Select a language** screen choose your desired language.
#. On the **Select your location** screen choose your country, territory or
   area.
#. On the **Configure the keyboard** screen use one of the available methods to
   choose your keyboard layout.
#. On the **Configure the network** screen specify your *hostname*.
#. On the **Choose a mirror of the Ubuntu archive** series of screens choose
   the desired Ubuntu mirror to download the packages required for the
   installation. The best mirror is usually the one geographically closest to
   you. If you need to specify an http proxy this should also be done in the
   respective screen (leave the prompt blank for no proxy).
#. On the **Set up users and passwords** series of screens specify the account
   details for systems default non-root user: full name, username, password
   and whether the user's home directory should be encrypted or not. This user
   will be able to use the ``sudo`` command to execute commands as ``root``.
#. On the **Configure the clock** confirm if the suggested time zone is correct
   or choose the desired one.
#. On the **Partition disks** series of screens choose the desired partitioning
   method, and the disk you use to partition for installation, doing if
   necessary any manual adjustments. In the end confirm your choices and write
   the new partitions to the disk.
#. On the **Configuring discovery** screen when asked how you want to manage
   upgrades on the system choose the option to install security updates
   automatically.
#. On the **Software selection** screen, choose only the **OpenSSH server** as
   the software to install.
#. On the **Install the GRUB boot loader on a hard disk** screen, when asked if
   you want to install the GRUB boot loader to the master boot record, answer
   **Yes**.
#. On the **Finish the installation** screen, when asked if the system clock is
   set to UTC, answer **Yes**.
#. The installation will now be finished. Follow the instruction on the screen
   to reboot the machine into your new installation.
