Intro
=====

This short guide presents one of many ways to structure ansible playbooks and
inventories for small fleets (i.e., file based inventory). The ideas presented
here have a strong focus on simplicity and maintainability. As a result the
document partially contradict the official `Best Practices`_ (read them
nevertheless).

Basic knowledge about ansible and its `Command Line Tools`_ is a prerequisite.
Also the `Glossary`_ is good help if terms used here are not clear.

.. _Best Practices: https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html
.. _Command Line Tools: https://docs.ansible.com/ansible/latest/user_guide/command_line_tools.html
.. _Glossary: https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html

Simple Playbook
===============

A playbook consists of a list of plays. A play specifies a set of hosts and a
list of tasks and handlers which are to be executed on the target hosts. The
following snippets are examples of valid (equivalent) playbooks:

.. literalinclude:: ../examples/01-minimal-play/no-roles.yml
   :language: yaml

.. literalinclude:: ../examples/01-minimal-play/roles-keyword.yml
   :language: yaml

.. literalinclude:: ../examples/01-minimal-play/import-role-task.yml
   :language: yaml

Note that the `roles` keyword is merely a shortcut for `import_role` tasks.
When the `roles` and `tasks` keywords are both specified in a play, then
`tasks` are executed after including roles (see the `Using Roles`_ of the users
guide).

.. Hint::

   **Do not use the roles keyword**

   In order to make the order of execution explicit, it is arguably better to
   just avoid the `roles` keyword altogether and use `include_role` inside
   `tasks` exclusively.

Official ansible documentation and large parts of the community alike often
stress the importance of organizing content into roles. The primary purpose of
roles is to group tasks, handlers, vars, defaults, files, templates, etc., into
packages which can be reused in other playbooks, across an organization and/or
published to `Galaxy`_. Using roles in playbooks regrettably add some
complexity. Especially:

* `Variable Precedence`_
* `Handler Execution`_ (GH Issue #10829)

.. Hint::

   **Do not create roles, extract them from playbooks**

   A typical infrastructure setup starts with a certain amount of organization
   specific boilerplate which is common across the fleet. The recommended
   `Directory Layout`_ even mentions a ``common`` role. However, creating roles
   with project / organization specific tasks somewhat contradicts their
   purpose and adds needless complexity.

   It is therefore advisable to always start out with a playbook containing a
   hand full of tasks. A playbook (or parts of it) can later be *extracted*
   into a role if necessary.


Tidy Inventory
==============

The ansible inventory is a list of hosts and host groups together with
associated variables generated from all inventory sources prior to every
playbook run.

If a directory is specified using the ``--inventory`` option when running one
of the ansible commands, then all the files found in that directory are merged
into one runtime inventory structure (See `Using multiple inventory sources`_
section in the users guide for details).

.. Tip:: Inspect the Inventory

   Use the ``ansible-inventory --list`` and ``ansible-inventory --host=example.com``
   commands to inspect the merged inventory including all variables specified
   by inventory sources. Try the ``--yaml`` flag for more readable output.

Ansible supports a whole lot of file formats and dynamic inventory sources out
of the box via `Inventory Plugins`_. Still, many examples feature single-file
INI-style inventories -- probably for historical reasons.

In addition to the inventory, ansible supports collecting variables from `Vars
Plugins`_ before executing playbooks. In fact, the `host_vars` and `group_vars`
constructs are implemented that way.

.. Hint::

   **Define variables in one place only**

   The flexibility of ansible sometimes gets people confused, especially when
   it comes to where variables are defined and maintained. Thus either define
   them in the inventory *or* via `host_vars` / `group_vars` directories. Never
   use both mechanisms in the same project.


Maintainable Project Structure
==============================

Simplicity is key when setting up ansible infra projects. Otherwise code
maintenance and troubleshooting efforts quickly eat up the benefits gained
through automation.

One of the problems the author observed in several ansible deployments is the
fact that variable definitions are littered all over the code base (inventory
files, group_vars, host_vars, role vars, role defaults, var files, etc.)

Using the following rules, this problem can be mitigated quite a bit:

1. For each playbook create a separate inventory file (YAML format)
2. Add a group with the name of the playbook in the inventory file
3. Specify default values as group variables in the inventory file
4. Add single hosts or subgroups to the playbook group
5. Override variable values in host entries / subgroup entries of the playbook group
6. Reference the playbook group of the dedicated inventory file from the ``hosts`` keyword in the playbook.

.. Important::

   * Do not use `host_vars` and `group_vars`, those are covered by the YAML
     inventory in this case.
   * The same variable must only be defined in one inventory file. Sticking to
     a naming pattern respecting (parts of) the file name helps.

Some variables are not related to specific playbooks. E.g., configuration which
specifies ssh connection parameters. Such parameters can be placed in a
separate YAML file inside the inventory directory (e.g., ``ansible.yml``).

Separate inventory files can also be added in order to specify host groups
which can be reused from playbook specific inventory files. For example if the
infra spans more than one location add a ``hosts_locations.yml`` file
specifying location groups and member machines.

An example setup might contain the following playbook related files:

::

   infra_time.yml
   inventory/playbook_infra_time.yml

   app_web.yml
   inventory/playbook_app_web.yml

In addition one global playbook (``site.yml``) and some inventory files
defining reusable host groups and specifying how ansible connects to some of
the machines:

::

   site.yml

   inventory/ansible.yml
   inventory/hosts_location.yml
   inventory/hosts.yml

See the `znerol/ansible-maintainable-playbooks`_ for the whole example.

.. _Directory Layout: https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout
.. _Galaxy: https://galaxy.ansible.com/docs/contributing/creating_role.html
.. _Handler Execution: https://github.com/ansible/ansible/issues/10829
.. _Inventory Plugins: https://docs.ansible.com/ansible/latest/plugins/inventory.html
.. _Using Roles: https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#using-roles
.. _Using multiple inventory sources: https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#using-multiple-inventory-sources
.. _Variable Precedence: https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable
.. _Vars Plugins: https://docs.ansible.com/ansible/latest/plugins/vars.html
.. _znerol/ansible-maintainable-playbooks: https://github.com/znerol/ansible-maintainable-playbooks
