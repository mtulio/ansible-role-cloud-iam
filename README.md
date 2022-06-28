cloud-iam
=========

![](https://github.com/mtulio/ansible-role-cloud-iam/actions/workflows/release.yml/badge.svg)
![](https://github.com/mtulio/ansible-role-cloud-iam/actions/workflows/ci.yml/badge.svg?branch=main)

Ansible role to manage **I**dentity and **A**ccess **M**anagement resources in your Cloud
Infrastructure, both Unix-based systems and Cloud service provider.

Now we are supporting AWS - please help us to improve =]

Requirements
------------

- boto3
- ansible >= 4.1

Role Variables
--------------

`iam_roles`: the list of roles to create on resources.

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

* Create groups in Unix systems and AWS:

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


* Create users in Unix systems and AWS:

      - hosts: servers
        vars_files:
          - vars/vault_pass.yml
          - vars/vault_ssh_keys.yml
        vars:
          iam_users:
            - name: marco
              full_name: 'Marco'
              password: "{{ vault_pass_md5_marco }}"
              providers:
                - unix
                - aws
              ssh_pub_key: "{{ lookup('file', playbook_dir'/files/ssh_keys/marco.pub') }}"
              groups:
                - admin

            - name: rundeck
              password: "{{ vault_pass_md5_rundeck }}"
              providers:
                - aws
              ssh_pub_key: "{{ lookup('file', playbook_dir'/files/ssh_keys/rundeck.pub') }}"
              ssh_priv_key: "{{ vault_ssh_key_rundeck }}"
              groups:
                - rundeck

        roles:
           - { role: cloud-iam.mtulio }

* Create and keep updated AWS IAM role:

      - hosts: localhost
        vars:
          iam_roles:
          - iam_name: "instance-role-myserver"
            providers:
              - aws
            iam_s3_policies:
              - service: s3
                bucket: mybucket_01
                mode: rw
                file_type: template
                file_path: aws-s3-policy-rw.json.j2
              - service: s3
                bucket: mybucket_02
                mode: ro
                file_type: template
                file_path: aws-s3-policy-ro.json.j2
            iam_resources_policies:
              - service: custom
                resource: ec2-describe
                mode: ro
                file_type: template
                file_path: aws-ec2-describe.json.j2
                version: '2012-10-17'

          - iam_name: "lambda-myFunction"
            providers:
              - aws
            iam_policy_type: file
            iam_policy_path: "aws-sts-assume-lambda.json"
            iam_managed_policies:
                - arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole
            iam_resources_policies:
              - service: dynamodb
                resource: myTable
                mode: rw
                region: us-east-1
                file_type: template
                file_path:aws-dynamodb-rw.json.j2
            iam_s3_policies:
              - service: s3
                bucket: functionState
                mode: rw
                file_type: template
                file_path: aws-s3-policy-rw.json.j2

          - iam_name: "instance-role-dns"
            providers:
              - aws
            iam_resources_policies:
              - service: r53
                resource: mydomain.internal
                mode: delete
                zone_id: Z1FBB4KJZQ20Y7
                file_type: template
                file_path: aws-r53-rw-rrset.json.j2


        roles:
           - { role: cloud-iam.mtulio }

License
-------

GPLv3

TODO
----

* AWS
  * supporting creation of custom IAM policy
* Supporting other Cloud providers
* IPA
  * support to create users on IPA/IdM

Author Information
------------------

[Marco Túlio R Braga](https://github.com/mtulio)
