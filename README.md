##### SUMMARY
This implements a plugin to enable rendering of templates using values from HashiCorp Consul and Vault key-value stores.
It uses consul-template binary, that should exist in $PATH on Ansible controller.
consul-template documentation: https://github.com/hashicorp/consul-template
vault documentation: https://www.vaultproject.io/docs/what-is-vault/index.html

Module renders template locally on Controller and then sends result file using Ansible core Copy module, using any defined options not used by consul_template. So it can use almost all of Copy options, backup and validation, as well as --check and --diff playbook arguments. All of this was tested.

Also consul-template allow use of environment variables in templates. So i have implemented use of task environment option.

##### ADDITIONAL INFORMATION
You can simply try to use this module. Download latest version of Vault and Consul-template binaries from HashiCorp official site (https://releases.hashicorp.com/) and place them in any location defined in your $PATH. Then follow instructions below.

Run Vault "dev mode" server with no need to configure it, then set required environment variable containing its address:
```
$ vault server --dev --log-level=warn &
$ export VAULT_ADDR=http://127.0.0.1:8200
```
Vault Dev Server documentation page: https://www.vaultproject.io/docs/concepts/dev-server.html

Put some test values into Vault:
```
$ vault kv put secret/testsecret secretkey=secretvalue
```

Put a template in file, for example, secret.vtmpl:
```
# secret.vtmpl:
{{ with secret "secret/testsecret" -}}
secretkey={{ index .Data.data "secretkey" }}
{{- end }}
```

Create a simple playbook which use consul_template module and example template:
```
# render.yml:
- hosts: localhost
  collections:
    - mmorev.hashicorp
  tasks:
  - consul_template:
      src: secret.vtmpl
      dest: /tmp/secret
```

Run play:
```
$ ansible-playbook render.yml
```

Check that ansible created a file `/tmp/secret` and it contains our secret value:
```
$ cat /tmp/secret 
secretkey=secretvalue
```

