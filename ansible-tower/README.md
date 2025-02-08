# Ansible Tower

![Ansible AWX Tower Logo](images/ansible-awx-tower-logo.png?raw=true "Ansible AWX Tower Logo")

## About

In this HashiQube DevOps Lab you will get hands on experience with Ansible AWX Tower. Ansible AWX Tower is a web interface for Ansible and it provides you with credentials, inventories, projects and playbooks. 

In addition you will learn how to configure AWX Tower using the CLI and trigger an Ansible Run from the command line. 

AWX provides a web-based user interface, REST API, and task engine built on top of Ansible. It is one of the upstream projects for Red Hat Ansible Automation Platform.

To install AWX, please view the Install guide https://github.com/ansible/awx/blob/devel/INSTALL.md

To learn more about using AWX, and Tower, view the Tower docs site http://docs.ansible.com/ansible-tower/index.html

With Red Hat® Ansible® Tower you can centralize and control your IT infrastructure with a visual dashboard, role-based access control, job scheduling, integrated notifications and graphical inventory management. Easily embed Ansible Tower into existing tools and processes with REST API and CLI.

## Provision

<!-- tabs:start -->

#### **Github Codespaces**
[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/star3am/hashiqube?quickstart=1)
```
bash docker/docker.sh
bash minikube/minikube.sh
bash ansible-tower/ansible-tower.sh
```

#### **Vagrant**

```
vagrant up --provision-with basetools,docker,docsify,minikube,ansible-tower
```

#### **Docker Compose**

```
docker compose exec hashiqube /bin/bash
bash bashiqube/basetools.sh
bash docker/docker.sh
bash docsify/docsify.sh
bash minikube/minikube.sh
bash ansible-tower/ansible-tower.sh
```

<!-- tabs:end -->

## Summary
After provision, you can access AWX Ansible Tower on http://localhost:8043 and login with User: __admin__ and the Password displayed at the end of the Provision operation.
![Ansible Tower](images/ansible-tower.png?raw=true "Ansible Tower")

See the Output below at the end of the Provision Operation, specifically the Login URL and Password
![Ansible Tower Password](images/ansible-tower-end-of-provision.png?raw=true "Ansible Tower Password")

## Run a playbook
In order to run a playbook, we have to do a few things, we are going to login to AWX Ansible Tower

Once logged in you will see this page

![Ansible Tower](images/ansible-tower-logged-in.png?raw=true "Ansible Tower")

## Add a project
:bulb: This was automatically done for you in the Provisioning step, with this command: 

```bash
# https://docs.ansible.com/ansible-tower/latest/html/towercli/reference.html#awx-projects-create
echo -e '\e[38;5;198m'"++++ "
echo -e '\e[38;5;198m'"++++ Create projects ansible-role-example-role"
echo -e '\e[38;5;198m'"++++ "
sudo --preserve-env=PATH -u vagrant /home/vagrant/.local/bin/awx projects create --organization 'Default' --scm_update_on_launch true --scm_url https://github.com/star3am/ansible-role-example-role --scm_type git --name ansible-role-example-role --description ansible-role-example-role --wait $AWX_COMMON
```

Now we can add a project, click on Projects in the menue on the left and add a new project, here is an example 
You can use my example repository https://github.com/star3am/ansible-role-example-role.git For the Source Control URL
![Ansible Tower](images/ansible-tower-add-project.png?raw=true "Ansible Tower")

## Add a Credential

:bulb: This was automatically done for you in the Provisioning Operation using this code: 

```bash
# https://docs.ansible.com/ansible-tower/latest/html/towercli/reference.html#awx-credentials-create
echo -e '\e[38;5;198m'"++++ "
echo -e '\e[38;5;198m'"++++ Add credentials ansible"
echo -e '\e[38;5;198m'"++++ "
sudo --preserve-env=PATH -u vagrant /home/vagrant/.local/bin/awx credentials create --credential_type 'Machine' --organization 'Default' --name 'ansible' --inputs '{"username": "vagrant", "password": "vagrant"}' $AWX_COMMON
```

Navigate to Credentials in the left menue and add a new Credential of type "Machine" select the default organisation and add username __vagrant__ and password __vagrant__
Ansible Tower will use these credentials to login to the Hashiqube VM when we run a job template.
![Ansible Tower](images/ansible-tower-add-credential.png?raw=true "Ansible Tower")

## Add Inventory

:bulb: This was automatically done for you in the Provisioning Operation using this code: 

```bash
# https://docs.ansible.com/ansible-tower/latest/html/towercli/reference.html#awx-inventory-list
echo -e '\e[38;5;198m'"++++ "
echo -e '\e[38;5;198m'"++++ Check if 'Demo Inventory' exists"
echo -e '\e[38;5;198m'"++++ "
sudo --preserve-env=PATH -u vagrant /home/vagrant/.local/bin/awx inventory list --wait $AWX_COMMON | grep -q 'Demo Inventory'
if [ $? -eq 1 ]; then
  echo -e '\e[38;5;198m'"++++ 'Demo Inventory' doesn't exist, creating"
  sudo --preserve-env=PATH -u vagrant /home/vagrant/.local/bin/awx inventory create --name 'Demo Inventory' --description 'Demo Inventory' --organization 'Default' --wait $AWX_COMMON
else
  echo -e '\e[38;5;198m'"++++ 'Demo Inventory' exists"
fi
```

Now let's add an Inventory, we will need this when we create the Job Template
Click on Inventories on the menue in the left and add a new Inventory.
![Ansible Tower](images/ansible-tower-add-inventory.png?raw=true "Ansible Tower")

## Add a Job Template

:bulb: This was automatically done for you in the Provisioning Operation using this code: 

```bash
# https://docs.ansible.com/ansible-tower/latest/html/towercli/reference.html#awx-job-templates-create
echo -e '\e[38;5;198m'"++++ "
echo -e '\e[38;5;198m'"++++ Create job_templates ansible-role-example-role"
echo -e '\e[38;5;198m'"++++ "
sudo --preserve-env=PATH -u vagrant /home/vagrant/.local/bin/awx job_templates create --name ansible-role-example-role --description ansible-role-example-role --job_type run --inventory 'Demo Inventory' --project 'ansible-role-example-role' --become_enabled true --ask_limit_on_launch true --ask_tags_on_launch true --playbook site.yml --ask_limit_on_launch true --ask_tags_on_launch true --ask_variables_on_launch true --wait $AWX_COMMON
```

Next we are going to add a Job Template, navigate to Templates in the menue on the left and add a new Job Template
Use the Inventory we created for Vagrant and the Project we created earlier. 

Select __site.yml__ for the Playbook.

Select the vagrant Credential we created earlier. 
![Ansible Tower](images/ansible-tower-add-template.png?raw=true "Ansible Tower")

Be sure to scroll down and select: 
- Privilege Escalation
- Provisioning Callbacks

Also supply a random string which we will use as the __Host Config Key__ `UL3H6uRtDozHA13trZudrUwUPBw4rSo7rRvi`
![Ansible Tower](images/ansible-tower-add-template-more.png?raw=true "Ansible Tower")

## Trigger a Run
Now we can login to Hashiqube and use this Callback to trigger an Ansible Run, let's do that
- SSH into Hashiqube by doing `vagrant ssh` in the project folder

![Ansible Tower](images/vagrant-ssh.png?raw=true "Ansible Tower")

Now let's use our Callback URL and the Host Config Key to trigger a run using Curl

__vagrant@hashiqube0:~$__ `curl -s -i -X POST -H Content-Type:application/json --data '{"host_config_key": "UL3H6uRtDozHA13trZudrUwUPBw4rSo7rRvi"}' https://10.9.99.10:8043/api/v2/job_templates/9/callback/ -v -k`
```log
*   Trying 10.9.99.10...
* TCP_NODELAY set
* Connected to 10.9.99.10 (10.9.99.10) port 8043 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Client hello (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use http/1.1
* Server certificate:
*  subject: C=US; ST=North Carolina; L=Durham; O=Ansible; OU=AWX Development; CN=awx.localhost
*  start date: Sep  2 22:46:29 2021 GMT
*  expire date: Sep  2 22:46:29 2022 GMT
*  issuer: C=US; ST=North Carolina; L=Durham; O=Ansible; OU=AWX Development; CN=awx.localhost
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
> POST /api/v2/job_templates/9/callback/ HTTP/1.1
> Host: 10.9.99.10:8043
> User-Agent: curl/7.58.0
> Accept: */*
> Content-Type:application/json
> Content-Length: 59
>
* upload completely sent off: 59 out of 59 bytes
< HTTP/1.1 201 Created
HTTP/1.1 201 Created
< Server: nginx
Server: nginx
< Date: Sun, 05 Sep 2021 03:23:16 GMT
Date: Sun, 05 Sep 2021 03:23:16 GMT
< Content-Length: 0
Content-Length: 0
< Connection: keep-alive
Connection: keep-alive
< Location: /api/v2/jobs/15/
Location: /api/v2/jobs/15/
< Vary: Accept, Accept-Language, Origin, Cookie
Vary: Accept, Accept-Language, Origin, Cookie
< Allow: GET, POST, HEAD, OPTIONS
Allow: GET, POST, HEAD, OPTIONS
< X-API-Product-Version: 19.3.0
X-API-Product-Version: 19.3.0
< X-API-Product-Name: AWX
X-API-Product-Name: AWX
< X-API-Node: awx_1
X-API-Node: awx_1
< X-API-Time: 0.261s
X-API-Time: 0.261s
< X-API-Query-Count: 87
X-API-Query-Count: 87
< X-API-Query-Time: 0.094s
X-API-Query-Time: 0.094s
< Content-Language: en
Content-Language: en
< X-API-Total-Time: 0.330s
X-API-Total-Time: 0.330s
< X-API-Request-Id: 1dd2fcdb1c2346b488d1b3ab85ae860e
X-API-Request-Id: 1dd2fcdb1c2346b488d1b3ab85ae860e
< Access-Control-Expose-Headers: X-API-Request-Id
Access-Control-Expose-Headers: X-API-Request-Id
< Strict-Transport-Security: max-age=15768000
Strict-Transport-Security: max-age=15768000

<
* Connection #0 to host 10.9.99.10 left intact
```

Back in Ansible Tower, click on __Jobs__ in the menue on the left
You should see a successful Job
![Ansible Tower](images/ansible-tower-jobs.png?raw=true "Ansible Tower")

And you can click on the job for more details
![Ansible Tower](images/ansible-tower-job-details.png?raw=true "Ansible Tower")

For Windows, let's create a Job Template
![Ansible Tower](images/ansible-tower-add-template-windows.png?raw=true "Ansible Tower")

Also be sure to tick: 
- Privilege Escalation
- Provisioning Callbacks

And for Windows we need to tell Ansible to use SSH and we need to specify the Shell
Pass the following in the extra variables section otherwise you will receive an error about Temp directory, see below
![Ansible Tower](images/ansible-tower-add-template-more-windows.png?raw=true "Ansible Tower")

```
ansible_shell_type: cmd
ansible_connection: ssh
```

![Ansible Tower](images/ansible-tower-job-details-windows-temp-dir-error.png?raw=true "Ansible Tower")

[google ads](../googleads.html ':include :type=iframe width=100% height=300px')

Let's use our Callback URL and the Host Config Key to trigger a run using Curl
           
__vagrant@ANSIBLE-ROLE-EX C:\Users\vagrant>__ `powershell.exe -Command "[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12;[System.Net.ServicePointManager]::ServerCertificateValidationCallback = { $true };Invoke-WebRequest -UseBasicParsing -Uri https://10.9.99.10:8043/api/v2/job_templates/10/callback/ -Method POST -Body @{host_config_key='UL3H6uRtDozHA13trZudrUwUPBw4rSo7rRvi'}"`
```log
StatusCode        : 201
StatusDescription : Created
Content           : {}
RawContent        : HTTP/1.1 201 Created
                    Connection: keep-alive
                    Vary: Accept, Accept-Language, Origin, Cookie
                    Allow: GET, POST, HEAD, OPTIONS
                    X-API-Product-Version: 19.3.0
                    X-API-Product-Name: AWX
                    X-API-Node: awx_1...
Headers           : {[Connection, keep-alive], [Vary, Accept, Accept-Language, Origin, Cookie], [Allow, GET, POST, HEAD, OPTIONS], [X-API-Product-Version,
                    19.3.0]...}
RawContentLength  : 0
```

Back in Ansible Tower, click on __Jobs__ in the menue on the left
You should see a successful Job
![Ansible Tower](images/ansible-tower-job-details-windows.png?raw=true "Ansible Tower")

## CLI
One thing I've always struggled with was feedback from Ansible Tower to a pipeline, for example how do we know if a run succeeded or failed.
Then I discovered AWX CLI and TOWER CLI and it can be installed with pip

Awx: `pip3 install awxkit`
Ansible Tower: `pip3 install ansible-tower-cli`

Using the Job template and inventory we created further up in the page we can now issue a run using AWX CLI

__vagrant@hashiqube0:~$__ `awx --conf.host https://10.9.99.10:8043 -f human job_templates launch 9 --monitor --filter status --conf.insecure --conf.username admin --conf.password password`
```log
------Starting Standard Out Stream------
[DEPRECATION WARNING]: COMMAND_WARNINGS option, the command warnings feature is
 being removed. This feature will be removed from ansible-core in version 2.14.
 Deprecation warnings can be disabled by setting deprecation_warnings=False in
ansible.cfg.
SSH password:

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host 10.9.99.10 should use
/usr/bin/python3, but is using /usr/bin/python for backward compatibility with
prior Ansible releases. A future Ansible release will default to using the
discovered platform python for this host. See https://docs.ansible.com/ansible-
core/2.11/reference_appendices/interpreter_discovery.html for more information.
 This feature will be removed in version 2.12. Deprecation warnings can be
disabled by setting deprecation_warnings=False in ansible.cfg.
ok: [10.9.99.10]

TASK [/runner/project : set_fact] **********************************************
skipping: [10.9.99.10]

TASK [/runner/project : set_fact] **********************************************
skipping: [10.9.99.10]

TASK [/runner/project : set_fact] **********************************************
skipping: [10.9.99.10]

TASK [/runner/project : set_fact] **********************************************
skipping: [10.9.99.10]

TASK [/runner/project : Cloud] *************************************************
skipping: [10.9.99.10]

TASK [/runner/project : OS] ****************************************************
skipping: [10.9.99.10]

TASK [/runner/project : Write Ansible hostvars to file] ************************
skipping: [10.9.99.10]

TASK [/runner/project : Enable EPEL Repository] ********************************
skipping: [10.9.99.10]

TASK [/runner/project : Ensure package manager repositories are configured | Get repo list] ***
skipping: [10.9.99.10]

TASK [/runner/project : Ensure package manager repositories are configured | Display repo list] ***
skipping: [10.9.99.10]

TASK [/runner/project : Get repo files list] ***********************************
skipping: [10.9.99.10]

TASK [/runner/project : Ensure package manager repositories are configured | Display repo list] ***
skipping: [10.9.99.10]

TASK [/runner/project : Install Package dependencies] **************************
skipping: [10.9.99.10] => (item=aide)
skipping: [10.9.99.10] => (item=ipset)
skipping: [10.9.99.10] => (item=firewalld)

TASK [/runner/project : FIX RHEL8-CIS SCORED | 1.4.1 | PATCH | Ensure X is installed] ***
skipping: [10.9.99.10] => (item=http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/aide-0.16-14.el8.x86_64.rpm)

TASK [/runner/project : FIX RHEL7-CIS AUTOMATED | 1.4.1 | PATCH | Ensure X is installed] ***
skipping: [10.9.99.10] => (item=http://mirror.centos.org/centos/7/os/x86_64/Packages/aide-0.15.1-13.el7.x86_64.rpm)

TASK [/runner/project : OS] ****************************************************
ok: [10.9.99.10] => {
    "msg": "Ubuntu 18.04 bionic on innotek GmbH"
}

TASK [/runner/project : set_fact] **********************************************
skipping: [10.9.99.10]

TASK [/runner/project : set_fact] **********************************************
skipping: [10.9.99.10]

TASK [/runner/project : set_fact] **********************************************
skipping: [10.9.99.10]

TASK [/runner/project : set_fact] **********************************************
ok: [10.9.99.10]

TASK [/runner/project : Write Ansible hostvars to file] ************************
changed: [10.9.99.10]

TASK [/runner/project : OS] ****************************************************
skipping: [10.9.99.10]

TASK [/runner/project : set_fact] **********************************************
skipping: [10.9.99.10]

TASK [/runner/project : set_fact] **********************************************
skipping: [10.9.99.10]

TASK [/runner/project : set_fact] **********************************************
skipping: [10.9.99.10]

TASK [/runner/project : set_fact] **********************************************
skipping: [10.9.99.10]

TASK [/runner/project : Write Ansible hostvars to file] ************************
skipping: [10.9.99.10]

PLAY RECAP *********************************************************************
10.9.99.10                 : ok=4    changed=1    unreachable=0    failed=0    skipped=24   rescued=0    ignored=0

------End of Standard Out Stream--------

status
==========
successful
```

## Terraform
So using the configuration above, let's use terraform to kick of an ansible run and display the output

We are going to use local-exec and remote-exec

```hcl
locals {
  timestamp = timestamp()
}

resource "null_resource" "awx_cli" {
  triggers = {
    timestamp = local.timestamp
  }

  provisioner "remote-exec" {
    inline = [
      "/home/vagrant/.local/bin/awx --conf.host https://10.9.99.10:8043 -f human job_templates launch 9 --monitor --filter status --conf.insecure --conf.username admin --conf.password password",
    ]

    connection {
      type        = "ssh"
      user        = "vagrant"
      password    = "vagrant"
      host        = "10.9.99.10"
    }
  }

  provisioner "local-exec" {
    command = "/usr/local/bin/awx --conf.host https://10.9.99.10:8043 -f human job_templates launch 9 --monitor --filter status --conf.insecure --conf.username admin --conf.password password"
  }
}
```

__~/workspace/hashiqube/ansible-tower(master*) »__ `terraform init`

```
Initializing the backend...

Initializing provider plugins...
- Using previously-installed hashicorp/null v3.1.0

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, we recommend adding version constraints in a required_providers block
in your configuration, with the constraint strings suggested below.

* hashicorp/null: version = "~> 3.1.0"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

__~/workspace/hashiqube/ansible-tower(master*) »__ `terraform apply --auto-approve`

```log
null_resource.awx_cli: Refreshing state... [id=2181705754889762329]
null_resource.awx_cli: Destroying... [id=2181705754889762329]
null_resource.awx_cli: Destruction complete after 0s
null_resource.awx_cli: Creating...
null_resource.awx_cli: Provisioning with 'remote-exec'...
null_resource.awx_cli (remote-exec): Connecting to remote host via SSH...
null_resource.awx_cli (remote-exec):   Host: 10.9.99.10
null_resource.awx_cli (remote-exec):   User: vagrant
null_resource.awx_cli (remote-exec):   Password: true
null_resource.awx_cli (remote-exec):   Private key: false
null_resource.awx_cli (remote-exec):   Certificate: false
null_resource.awx_cli (remote-exec):   SSH Agent: true
null_resource.awx_cli (remote-exec):   Checking Host Key: false
null_resource.awx_cli (remote-exec): Connected!
null_resource.awx_cli (remote-exec): ------Starting Standard Out Stream------
null_resource.awx_cli (remote-exec): ansible-playbook [core 2.11.5.post0]
null_resource.awx_cli (remote-exec):   config file = /runner/project/ansible.cfg
null_resource.awx_cli (remote-exec):   configured module search path = ['/home/runner/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules', '/runner/project/library']
null_resource.awx_cli (remote-exec):   ansible python module location = /usr/local/lib/python3.8/site-packages/ansible
null_resource.awx_cli (remote-exec):   ansible collection location = /runner/requirements_collections:/home/runner/.ansible/collections:/usr/share/ansible/collections
null_resource.awx_cli (remote-exec):   executable location = /usr/local/bin/ansible-playbook
null_resource.awx_cli (remote-exec):   python version = 3.8.6 (default, Jan 29 2021, 17:38:16) [GCC 8.4.1 20200928 (Red Hat 8.4.1-1)]
null_resource.awx_cli (remote-exec):   jinja version = 2.10.3
null_resource.awx_cli (remote-exec):   libyaml = True
null_resource.awx_cli (remote-exec): Using /runner/project/ansible.cfg as config file
null_resource.awx_cli (remote-exec): [DEPRECATION WARNING]: COMMAND_WARNINGS option, the command warnings feature is
null_resource.awx_cli (remote-exec):  being removed. This feature will be removed from ansible-core in version 2.14.
null_resource.awx_cli (remote-exec):  Deprecation warnings can be disabled by setting deprecation_warnings=False in
null_resource.awx_cli (remote-exec): ansible.cfg.
null_resource.awx_cli (remote-exec): SSH password:
null_resource.awx_cli: Still creating... [10s elapsed]
null_resource.awx_cli (remote-exec): statically imported: /runner/project/tasks/el.yml
null_resource.awx_cli (remote-exec): statically imported: /runner/project/tasks/deb.yml
null_resource.awx_cli (remote-exec): statically imported: /runner/project/tasks/windows.yml
null_resource.awx_cli (remote-exec): Skipping callback 'awx_display', as we already have a stdout callback.
null_resource.awx_cli (remote-exec): Skipping callback 'default', as we already have a stdout callback.
null_resource.awx_cli (remote-exec): Skipping callback 'minimal', as we already have a stdout callback.
null_resource.awx_cli (remote-exec): Skipping callback 'oneline', as we already have a stdout callback.

null_resource.awx_cli (remote-exec): PLAYBOOK: site.yml *************************************************************
null_resource.awx_cli (remote-exec): 1 plays in site.yml

null_resource.awx_cli (remote-exec): PLAY [all] *********************************************************************

null_resource.awx_cli (remote-exec): TASK [Gathering Facts] *********************************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/site.yml:2
null_resource.awx_cli (remote-exec): [DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host 10.9.99.10 should use
null_resource.awx_cli (remote-exec): /usr/bin/python3, but is using /usr/bin/python for backward compatibility with
null_resource.awx_cli (remote-exec): prior Ansible releases. A future Ansible release will default to using the
null_resource.awx_cli (remote-exec): discovered platform python for this host. See https://docs.ansible.com/ansible-
null_resource.awx_cli (remote-exec): core/2.11/reference_appendices/interpreter_discovery.html for more information.
null_resource.awx_cli (remote-exec):  This feature will be removed in version 2.12. Deprecation warnings can be
null_resource.awx_cli (remote-exec): disabled by setting deprecation_warnings=False in ansible.cfg.
null_resource.awx_cli (remote-exec): ok: [10.9.99.10]
null_resource.awx_cli (remote-exec): META: ran handlers

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:9
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:14
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:19
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:24
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : Cloud] *************************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:29
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {}

null_resource.awx_cli (remote-exec): TASK [/runner/project : OS] ****************************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:33
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {}

null_resource.awx_cli (remote-exec): TASK [/runner/project : Write Ansible hostvars to file] ************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:37
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : Enable EPEL Repository] ********************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:46
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : Ensure package manager repositories are configured | Get repo list] ***
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:56
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : Ensure package manager repositories are configured | Display repo list] ***
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:65
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {}

null_resource.awx_cli (remote-exec): TASK [/runner/project : Get repo files list] ***********************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:71
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : Ensure package manager repositories are configured | Display repo list] ***
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:75
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {}

null_resource.awx_cli (remote-exec): TASK [/runner/project : Install Package dependencies] **************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:81
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => (item=aide)  => {"ansible_loop_var": "item", "changed": false, "item": "aide", "skip_reason": "Conditional result was False"}
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => (item=ipset)  => {"ansible_loop_var": "item", "changed": false, "item": "ipset", "skip_reason": "Conditional result was False"}
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => (item=firewalld)  => {"ansible_loop_var": "item", "changed": false, "item": "firewalld", "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : FIX RHEL8-CIS SCORED | 1.4.1 | PATCH | Ensure X is installed] ***
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:102
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => (item=http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/aide-0.16-14.el8.x86_64.rpm)  => {"ansible_loop_var": "item", "changed": false, "item": "http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/aide-0.16-14.el8.x86_64.rpm", "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : FIX RHEL7-CIS AUTOMATED | 1.4.1 | PATCH | Ensure X is installed] ***
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/el.yml:115
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => (item=http://mirror.centos.org/centos/7/os/x86_64/Packages/aide-0.15.1-13.el7.x86_64.rpm)  => {"ansible_loop_var": "item", "changed": false, "item": "http://mirror.centos.org/centos/7/os/x86_64/Packages/aide-0.15.1-13.el7.x86_64.rpm", "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : OS] ****************************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/deb.yml:8
null_resource.awx_cli (remote-exec): ok: [10.9.99.10] => {
null_resource.awx_cli (remote-exec):     "msg": "Ubuntu 18.04 bionic on innotek GmbH"
null_resource.awx_cli (remote-exec): }

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/deb.yml:12
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/deb.yml:17
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/deb.yml:22
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/deb.yml:27
null_resource.awx_cli (remote-exec): ok: [10.9.99.10] => {"ansible_facts": {"cloud": "vagrant"}, "changed": false}

null_resource.awx_cli (remote-exec): TASK [/runner/project : Write Ansible hostvars to file] ************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/deb.yml:32
null_resource.awx_cli (remote-exec): changed: [10.9.99.10] => {"changed": true, "checksum": "9a8877d65d2ea053b348fd733e0b35c3fe67587e", "dest": "/soe-20210703201014.json", "gid": 0, "group": "root", "md5sum": "da8cfcb796842fa90cfa200cab4e05fa", "mode": "0644", "owner": "root", "size": 77371, "src": "/tmp/ansible-tmp-1632275535.6399148-87-109361542358393/source", "state": "file", "uid": 0}

null_resource.awx_cli (remote-exec): TASK [/runner/project : OS] ****************************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/windows.yml:8
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {}

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/windows.yml:12
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/windows.yml:17
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/windows.yml:22
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/windows.yml:27
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (remote-exec): TASK [/runner/project : Write Ansible hostvars to file] ************************
null_resource.awx_cli (remote-exec): task path: /runner/project/tasks/windows.yml:32
null_resource.awx_cli (remote-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}
null_resource.awx_cli (remote-exec): META: role_complete for 10.9.99.10
null_resource.awx_cli (remote-exec): META: ran handlers
null_resource.awx_cli (remote-exec): META: ran handlers

null_resource.awx_cli (remote-exec): PLAY RECAP *********************************************************************
null_resource.awx_cli (remote-exec): 10.9.99.10                 : ok=4    changed=1    unreachable=0    failed=0    skipped=24   rescued=0    ignored=0

null_resource.awx_cli (remote-exec): ------End of Standard Out Stream--------
null_resource.awx_cli (remote-exec):
null_resource.awx_cli (remote-exec): status
null_resource.awx_cli (remote-exec): ==========
null_resource.awx_cli (remote-exec): successful
null_resource.awx_cli: Provisioning with 'local-exec'...
null_resource.awx_cli (local-exec): Executing: ["/bin/sh" "-c" "/usr/local/bin/awx --conf.host https://10.9.99.10:8043 -f human job_templates launch 9 --monitor --filter status --conf.insecure --conf.username admin --conf.password password"]
null_resource.awx_cli: Still creating... [20s elapsed]
null_resource.awx_cli: Still creating... [30s elapsed]
null_resource.awx_cli (local-exec): ------Starting Standard Out Stream------
null_resource.awx_cli (local-exec): ansible-playbook [core 2.11.5.post0]
null_resource.awx_cli (local-exec):   config file = /runner/project/ansible.cfg
null_resource.awx_cli (local-exec):   configured module search path = ['/home/runner/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules', '/runner/project/library']
null_resource.awx_cli (local-exec):   ansible python module location = /usr/local/lib/python3.8/site-packages/ansible
null_resource.awx_cli (local-exec):   ansible collection location = /runner/requirements_collections:/home/runner/.ansible/collections:/usr/share/ansible/collections
null_resource.awx_cli (local-exec):   executable location = /usr/local/bin/ansible-playbook
null_resource.awx_cli (local-exec):   python version = 3.8.6 (default, Jan 29 2021, 17:38:16) [GCC 8.4.1 20200928 (Red Hat 8.4.1-1)]
null_resource.awx_cli (local-exec):   jinja version = 2.10.3
null_resource.awx_cli (local-exec):   libyaml = True
null_resource.awx_cli (local-exec): Using /runner/project/ansible.cfg as config file
null_resource.awx_cli (local-exec): [DEPRECATION WARNING]: COMMAND_WARNINGS option, the command warnings feature is
null_resource.awx_cli (local-exec):  being removed. This feature will be removed from ansible-core in version 2.14.
null_resource.awx_cli (local-exec):  Deprecation warnings can be disabled by setting deprecation_warnings=False in
null_resource.awx_cli (local-exec): ansible.cfg.
null_resource.awx_cli (local-exec): SSH password:
null_resource.awx_cli (local-exec): statically imported: /runner/project/tasks/el.yml
null_resource.awx_cli (local-exec): statically imported: /runner/project/tasks/deb.yml
null_resource.awx_cli (local-exec): statically imported: /runner/project/tasks/windows.yml
null_resource.awx_cli (local-exec): Skipping callback 'awx_display', as we already have a stdout callback.
null_resource.awx_cli (local-exec): Skipping callback 'default', as we already have a stdout callback.
null_resource.awx_cli (local-exec): Skipping callback 'minimal', as we already have a stdout callback.
null_resource.awx_cli (local-exec): Skipping callback 'oneline', as we already have a stdout callback.

null_resource.awx_cli (local-exec): PLAYBOOK: site.yml *************************************************************
null_resource.awx_cli (local-exec): 1 plays in site.yml

null_resource.awx_cli (local-exec): PLAY [all] *********************************************************************

null_resource.awx_cli (local-exec): TASK [Gathering Facts] *********************************************************
null_resource.awx_cli (local-exec): task path: /runner/project/site.yml:2
null_resource.awx_cli (local-exec): [DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host 10.9.99.10 should use
null_resource.awx_cli (local-exec): /usr/bin/python3, but is using /usr/bin/python for backward compatibility with
null_resource.awx_cli (local-exec): prior Ansible releases. A future Ansible release will default to using the
null_resource.awx_cli (local-exec): discovered platform python for this host. See https://docs.ansible.com/ansible-
null_resource.awx_cli (local-exec): core/2.11/reference_appendices/interpreter_discovery.html for more information.
null_resource.awx_cli (local-exec):  This feature will be removed in version 2.12. Deprecation warnings can be
null_resource.awx_cli (local-exec): disabled by setting deprecation_warnings=False in ansible.cfg.
null_resource.awx_cli (local-exec): ok: [10.9.99.10]
null_resource.awx_cli (local-exec): META: ran handlers

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:9
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:14
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:19
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:24
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : Cloud] *************************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:29
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {}

null_resource.awx_cli (local-exec): TASK [/runner/project : OS] ****************************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:33
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {}

null_resource.awx_cli (local-exec): TASK [/runner/project : Write Ansible hostvars to file] ************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:37
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : Enable EPEL Repository] ********************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:46
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : Ensure package manager repositories are configured | Get repo list] ***
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:56
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : Ensure package manager repositories are configured | Display repo list] ***
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:65
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {}

null_resource.awx_cli (local-exec): TASK [/runner/project : Get repo files list] ***********************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:71
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : Ensure package manager repositories are configured | Display repo list] ***
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:75
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {}

null_resource.awx_cli (local-exec): TASK [/runner/project : Install Package dependencies] **************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:81
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => (item=aide)  => {"ansible_loop_var": "item", "changed": false, "item": "aide", "skip_reason": "Conditional result was False"}
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => (item=ipset)  => {"ansible_loop_var": "item", "changed": false, "item": "ipset", "skip_reason": "Conditional result was False"}
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => (item=firewalld)  => {"ansible_loop_var": "item", "changed": false, "item": "firewalld", "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : FIX RHEL8-CIS SCORED | 1.4.1 | PATCH | Ensure X is installed] ***
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:102
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => (item=http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/aide-0.16-14.el8.x86_64.rpm)  => {"ansible_loop_var": "item", "changed": false, "item": "http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/aide-0.16-14.el8.x86_64.rpm", "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : FIX RHEL7-CIS AUTOMATED | 1.4.1 | PATCH | Ensure X is installed] ***
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/el.yml:115
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => (item=http://mirror.centos.org/centos/7/os/x86_64/Packages/aide-0.15.1-13.el7.x86_64.rpm)  => {"ansible_loop_var": "item", "changed": false, "item": "http://mirror.centos.org/centos/7/os/x86_64/Packages/aide-0.15.1-13.el7.x86_64.rpm", "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : OS] ****************************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/deb.yml:8
null_resource.awx_cli (local-exec): ok: [10.9.99.10] => {
null_resource.awx_cli (local-exec):     "msg": "Ubuntu 18.04 bionic on innotek GmbH"
null_resource.awx_cli (local-exec): }

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/deb.yml:12
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/deb.yml:17
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/deb.yml:22
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/deb.yml:27
null_resource.awx_cli (local-exec): ok: [10.9.99.10] => {"ansible_facts": {"cloud": "vagrant"}, "changed": false}

null_resource.awx_cli (local-exec): TASK [/runner/project : Write Ansible hostvars to file] ************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/deb.yml:32
null_resource.awx_cli (local-exec): changed: [10.9.99.10] => {"changed": true, "checksum": "eea47aeb336e4b507f8dc851b5999f04fdc3d5aa", "dest": "/soe-20210703201014.json", "gid": 0, "group": "root", "md5sum": "c09a9e6b638526aea9d83a7d30c03d58", "mode": "0644", "owner": "root", "size": 77371, "src": "/tmp/ansible-tmp-1632275552.8567505-87-55845101901960/source", "state": "file", "uid": 0}

null_resource.awx_cli (local-exec): TASK [/runner/project : OS] ****************************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/windows.yml:8
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {}

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/windows.yml:12
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/windows.yml:17
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/windows.yml:22
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : set_fact] **********************************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/windows.yml:27
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}

null_resource.awx_cli (local-exec): TASK [/runner/project : Write Ansible hostvars to file] ************************
null_resource.awx_cli (local-exec): task path: /runner/project/tasks/windows.yml:32
null_resource.awx_cli (local-exec): skipping: [10.9.99.10] => {"changed": false, "skip_reason": "Conditional result was False"}
null_resource.awx_cli (local-exec): META: role_complete for 10.9.99.10
null_resource.awx_cli (local-exec): META: ran handlers
null_resource.awx_cli (local-exec): META: ran handlers

null_resource.awx_cli (local-exec): PLAY RECAP *********************************************************************
null_resource.awx_cli (local-exec): 10.9.99.10                 : ok=4    changed=1    unreachable=0    failed=0    skipped=24   rescued=0    ignored=0

null_resource.awx_cli (local-exec): ------End of Standard Out Stream--------
null_resource.awx_cli (local-exec):
null_resource.awx_cli (local-exec): status
null_resource.awx_cli (local-exec): ==========
null_resource.awx_cli (local-exec): successful
null_resource.awx_cli: Creation complete after 36s [id=936123805330159798]

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
```

## Links 

https://github.com/ansible/awx <br />
https://www.ansible.com/blog/ansible-tower-feature-spotlight-custom-credentials <br />
https://github.com/ybalt/ansible-tower <br />
https://www.ansible.com/products/tower <br />
https://www.ansible.com/ <br />

## Ansible AWX Tower

[filename](ansible-tower.sh ':include :type=code')

## Terraform and AWX

[filename](main.tf ':include :type=code hcl')

[google ads](../googleads.html ':include :type=iframe width=100% height=300px')