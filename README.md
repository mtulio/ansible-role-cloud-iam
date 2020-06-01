cloud-iam
=========

Ansible role to manage **I**dentity and **A**ccess **M**anagement resources in your Cloud
Infrastructure, both Unix-based systems and Cloud service provider - Now we are
supporting AWS (please help us to improve =] )


Requirements
------------

- boto3
- ansible >= 2.3

Role Variables
--------------

`iam_groups`: Groups to be created or removed, depending on the `state` option.

- `name`: Group's name
- `providers`: list of cloud provider. Supported: `unix` and `aws`
- `unix_sudoers_line`: list of sudoers line itens. `value` should be sudoers file syntax. `regex` should match to unique line on /etc/sudoers.

`iam_user`: Users to be created or removed, depending on the `state` option.

- `name`: User's name
- `full_name`: Full Name of the user
- `providers`: list of cloud provider. Supported: `unix` and `aws`
- `ssh_pub_key`: SSH public key to be added on `unix` provider.
- `groups`: groups to be associated to the user.

Dependencies
------------

`boto` and `boto3`: when using `aws` provider.

Example Playbook
----------------

* Create Groups:

      - hosts: servers
        vars:
          iam_groups:
            - name: admin
              providers:
                - unix
                - aws
              unix_sudoers_line:
                - regex: '^%admin'
                  value: '%admin ALL=(ALL) NOPASSWD: ALL'
              aws_managed_policies:
                - arn:aws:iam::aws:policy/IAMFullAccess
                - arn:aws:iam::aws:policy/job-function/Billing
                - arn:aws:iam::aws:policy/AdministratorAccess
                - arn:aws:iam:::policy/pol-BillingFullAccess
            - name: rundeck
              providers:
                - unix
              unix_sudoers_line:
                - regex: '^%rundeck'
                  value: '%rundeck ALL=(ALL) NOPASSWD: ALL'

        roles:
           - { role: cloud-iam.mtulio }

* Create groups in Unix systems and AWS:

      - hosts: servers
        vars:
          iam_users:
            - name: marcobraga
              full_name: 'Marco Braga'
              password: "{{ pass_md5_marcobraga }}"
              providers:
                - unix
                - aws
              ssh_pub_key: "{{ ssh_pub_marcobraga }}"
              groups:
                - admin

            - name: rundeck
              password: "{{ pass_md5_rundeck }}"
              providers:
                - aws
              ssh_pub_key: "{{ ssh_pub_rundeck }}"
              ssh_priv_key: "{{ ssh_key_rundeck }}"
              groups:
                - rundeck

        roles:
           - { role: cloud-iam.mtulio }

License
-------

GPLv3

Author Information
------------------

[Marco Túlio R Braga](https://github.com/mtulio)
