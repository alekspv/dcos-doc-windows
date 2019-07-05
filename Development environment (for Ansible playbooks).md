## Development environment (for Ansible playbooks):
_____________
**You need https://github.com/alekspv/examples/tree/feature/windows-support/aws/windows-agent**
_____________

Edit next lines in the file **Terraformfile** 

```
"dcos-terraform/dcos-install-remote-exec-ansible/null": {
-    "source": "git::https://github.com/dcos-terraform/terraform-null-dcos-install-remote-exec-ansible.git?ref=feature/windows-support"
  }
}
```
```
"dcos-terraform/dcos-install-remote-exec-ansible/null": {
+    "source": "git::https://github.com/OleksandrBielov/terraform-null-dcos-install-remote-exec-ansible.git?ref=feature/windows-support"
  }
}
```

```
terraform init -upgrade
terraform apply
```

terraform module (creating docker & run Ansible on bootstrap-node)

https://github.com/OleksandrBielov/terraform-null-dcos-install-remote-exec-ansible/tree/feature/windows-support

Our Ansible-playbook

https://github.com/alekspv/dcos-ansible/tree/feature/windows
