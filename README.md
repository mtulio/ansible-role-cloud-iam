cloud-iam
=========

Ansible role to manage Identity and Access Management resources in your Cloud
Infrastructure, both Unix-based systems and Cloud service provider.

Now we are supporting AWS - please help us to improve =]


Requirements
------------

- boto3
- ansible >= 2.3

Role Variables
--------------

`iam_roles`: the list of roles to create on resources.

`iam_groups`: the list of groups to create on resources.

`iam_users`: the list of users to create on resources.

Dependencies
------------

None

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

[Marco Tulio R Braga](https://github.com/mtulio)
