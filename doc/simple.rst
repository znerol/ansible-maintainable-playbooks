Simple playbook
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

.. Hint:: Do not use the roles keyword

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

.. Hint:: Do not create roles, extract them from playbooks

   A typical infrastructure setup starts with a certain amount of organization
   specific boilerplate which is common across the fleet. The recommended
   `Directory Layout`_ even mentions a ``common`` role. However, creating roles
   with project / organization specific tasks somewhat contradicts their
   purpose and adds needless complexity.

   It is therefore advisable to always start out with a playbook containing a
   hand full of tasks. A playbook (or parts of it) can later be *extracted*
   into a role if necessary.

.. _Directory Layout: https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout
.. _Galaxy: https://galaxy.ansible.com/docs/contributing/creating_role.html
.. _Handler Execution: https://github.com/ansible/ansible/issues/10829
.. _Using Roles: https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#using-roles
.. _Variable Precedence: https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable
