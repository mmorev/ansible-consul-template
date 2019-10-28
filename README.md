##### SUMMARY
This implements a plugin to enable rendering of templates using values from HashiCorp Consul and Vault key-value stores.
It uses consul-template binary, that should exist in $PATH on Ansible controller.

[consul-template GitHub page](https://github.com/hashicorp/consul-template)

[Vault documentation](https://www.vaultproject.io/docs/what-is-vault/index.html)

Module renders template locally on controller and then sends result file using ansible core copy module, using any defined options not used by consul_template. so it can use almost all of copy options, backup and validation, as well as --check and --diff playbook arguments. all of this was tested.

Also consul-template allow use of environment variables in templates. so i have implemented use of task environment option.

##### Additional information
You can simply try to use this module. download latest version of vault and consul-template binaries from [HashiCorp official site](https://releases.hashicorp.com/) and place them in any location defined in your $path. then follow instructions below.

run vault and consul in "dev mode" with no need to configure them, then set environment variable containing vault address:
```
$ vault server --dev --log-level=warn &
$ consul agent -dev -log-level=warn &
$ export vault_addr=http://127.0.0.1:8200
```
[Vault dev server documentation page](https://www.vaultproject.io/docs/concepts/dev-server.html)
[Consul dev server documentation page](https://learn.hashicorp.com/consul/getting-started/agent)

put some test values into vault:
```
$ vault kv put secret/testsecret secretkey=secretvalue
```

put a template in file, for example, secret.vtmpl:
```
# secret.vtmpl:
# Vault query example:
{{ with secret "secret/testsecret" -}}
secretkey={{ index .Data.data "secretkey" }}
{{- end }}
# Consul query example:
openkey={{ key "openkey" }}
```

create a simple playbook which use consul_template module and example template:
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

run play:
```
$ ansible-playbook render.yml
```

check that ansible created a file `/tmp/secret` and it contains our secret value:
```
$ cat /tmp/secret 
secretkey=secretvalue
```

