.. title: Server Configuration Management With Chef solo
.. slug: server-configuration-management-with-chef-solo
.. date: 2012-09-15
.. tags: blog
.. author: Pedro Romano
.. link:
.. description:
.. category: linux, ubuntu, chef

Server Configuration Management With Chef Solo
==============================================

Having set up one Linux server from scratch too many, I decided to investigate
`configuration management
<https://en.wikipedia.org/wiki/Configuration_management>`_ solutions and try to
automatise my setups.

I ended up choosing `Chef Solo
<http://wiki.opscode.com/display/chef/Chef+Solo>`_ for the task, due to being a
standalone solution not requiring a client/server setup. The following
blog posts served as inspiration and reference for my implementation:

* `Chef Solo tutorial: Managing a single server with Chef
  <http://www.opinionatedprogrammer.com/2011/06/chef-solo-tutorial-managing-a-single-server-with-chef/>`_
* `Simple Chef Solo Tutorial
  <http://illuminatedcomputing.com/posts/2012/02/simple-chef-solo-tutorial/>`_

Following up my :doc:`previous post <minimal-ubuntu-precise>` where
I set up a base minimum Ubuntu installation with a *SSH* daemon running on the
server (a requirement for my solution), I will now describe my implementation.

`Revision control <https://en.wikipedia.org/wiki/Revision_control>`_ is a must
for any project, so my first step was to create a `git
<https://en.wikipedia.org/wiki/Git_%28software%29>`_ `repository
<https://github.com/pmcnr/digitaluna-chef>`_ on `github <https://github.com/>`_
to maintain the *Chef* cookbooks and recipes as well as any additional files
required by the solution. This also has the added advantage that cloning the
git repository is an excellent way of deploying these files on the server.

The first component of the solution is a *bash* `bootstrap script
<https://github.com/pmcnr/digitaluna-chef/blob/master/bootstrap.sh>`_ which
runs on the local workstation, taking as its argument the remote user (which
will need to have ``sudo`` privileges) and server address in the
``user@remote-ip-or-fqdn`` format, and through a *SSH* session takes care of
automatising the following steps:

* Add the local user's public *SSH* key as a remote user's authorised key so
  the remote user's password doesn't have to be entered again (this requires
  typing the remote user's password).
* Update the remote system's package index and upgrade the installed packages
  to the latest available version.
* Install the *git* package and its dependencies.
* Clone the *git* `repository`_ and its sub-modules (not wanting to re-invent
  the wheel I make use of some `Opscode Public Cookbooks
  <https://github.com/opscode-cookbooks>`_ which are referenced in the *git*
  repository as sub-modules).
* Hand over control to a second *bash* `installation script
  <https://github.com/pmcnr/digitaluna-chef/blob/master/install.sh>`_ that is
  available in the cloned repository.

Running on the remote server, this `installation script`_ automatises, in turn,
the following steps:

* Install the ``debconf-utils`` and ``python-software-properties`` system
  packages which simplify the tasks of scripting the installation of packages
  with interactive prompts and of adding additional package repositories from
  the command line (or in this case from a script).
* Add the `OpsCode APT repository <http://apt.opscode.com/>`_ to the system's
  package repositories, from which we'll be able to install the *Chef*
  packages.
* Install the `chef` package and all its dependencies (the `chef` package
  includes *Chef Solo*), and create a link from its configuration file in our
  repository to its default location at ``/etc/chef``.

Finally, with *Chef Solo* installed and configured, and with many thanks to our
*Bash* scripts which took care of bootstrapping our *Chef* installation, we can
now start using *Chef* recipies for all the remaining steps! The last line of
the `installation script`_ simply invokes `chef-solo` with our default *Chef*
`JSON <https://en.wikipedia.org/wiki/JSON>`_ `configuration file
<https://github.com/pmcnr/digitaluna-chef/blob/master/solo.json>`_.

The *Chef* recipes perform (as they should) the bulk of the server installation
and configuration (the previous steps were just the minimum required to install
and configure *Chef*). The following are the actions performed by the *Chef*
recipes:

* Disable running the ``chef`` client service on start-up. Since we will be
  running *Chef Solo* and aren't really connected to a *Chef* server and so
  having the *Chef* client running would be redundant and wasteful.
* Install the `etckeeper <http://joeyh.name/code/etckeeper/>`_ package: keeping
  on with the *revision control everywhere* mantra, having the server's
  configuration files under ``/etc`` revision controlled seems like a very good
  idea and `etckeeper`_ does this automatically for you! The recipe also tweaks
  the `etckeeper`_ configuration to use `git`_ (which we already have
  installed) instead of `Bazaar
  <https://en.wikipedia.org/wiki/Bazaar_%28software%29>`_ which is the default
  for the default `DRCS
  <https://en.wikipedia.org/wiki/Distributed_revision_control>`_ for Ubuntu
  (although the author prefers `git`_ in the source distribution and recommends
  its use).
* Install the `ufw <http://en.wikipedia.org/wiki/Uncomplicated_Firewall>`_
  *(Uncomplicated Firewall)* package. The default recipe blocks all incoming
  ports with the exception of the *SSH* port (22).
* Reconfigure the *SSH* daemon to disallow root logins and password
  authentication (it will only allow connections authenticated with a public
  key for non-root users). This and the previous steps are obvious security
  best practices.
* Install the `acpid` daemon package to allow host initiated graceful shutdowns
  on KVM virtual machine guests (which is the case of the this system).
* Install the `byobu <http://en.wikipedia.org/wiki/Byobu_%28software%29)>`_
  package to provide an enhanced terminal multiplexer experience when
  interactive sessions with the server are required.
* Install `vim <https://en.wikipedia.org/wiki/Vim_%28text_editor%29>`_.
  Although I belong to the `Church of Emacs
  <https://en.wikipedia.org/wiki/Editor_war#Humor>`_, I am not a
  fundamentalist, and, because of its ubiquity, I tend to use an editor of the
  `vi <https://en.wikipedia.org/wiki/Vi>`_ persuasion when wearing my systems
  administrator hat.
* Install `htop <http://htop.sourceforge.net/>`_ for process and system
  resources monitoring.
* Install `multitail <http://www.vanheusden.com/multitail/>`_ *(tail on
  steroids!)* for an improved experience when following log files.
* Finally install the `Apache2 <https://httpd.apache.org/>`_ web server (using
  the default recipe from the `Opscode public
  cookbook <https://github.com/opscode-cookbooks/apache2>`_) and reconfigure the
  firewall to allow HTTP (port 80) traffic in.

And that sums it up! A functional *Ubuntu 12.04 LTS* system with an *Apache2*
HTTP server setup, fully automatised using *Chef Solo* and a couple of
bootstrap shell scripts.
