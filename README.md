# ansible-for-splunk-cloud

## Usage

:warning: Warning: These ansible playbooks will adjust configurations in your Splunk Cloud environment. In this process configurations like apps or indexes will get deleted when not configured in ansible correctly.

### Quickstart

1. Add your Splunk Cloud FQDN to `inventory/inventory.yml`.
2. Generate a token with an user which has the [required capabilities](https://docs.splunk.com/Documentation/SplunkCloud/latest/Config/ACSreqs). The `sc_admin` role has all required capabilities by default.
3. Encrypt the token with `ansible-vault` and place it into `inventory/*/host_vars/vars.yml`.
4. Encrypt your splunkbase username + password and place it into `inventory/group_vars/splunkcloud.yml`
5. Use GitHub repositiory secrets to store the ansible-vault password in the `ANSIBLE_VAULT_PASSWORD` variable (this is needed so the workflows can decrypt the variables when running the playbooks).
6. Adjust the config files to trigger the workflows/ansible playbooks.

### create/update/delete an index

```bash
ansible-playbook -i inventory --ask-vault-password cloud_index_config.yml

# only create/update indexes without removing anything
ansible-playbook -i inventory --ask-vault-password cloud_index_config.yml --skip-tags splunk_cloud_remove_index
```

```yaml
splunk_cloud_index_list:
  - name: "testindex"
    searchableDays: 92
    maxDataSizeMB: 1024
  - name: "testindex-2"
    datatype: "metric"
```

### install/update/remove a private app

```bash
ansible-playbook -i inventory --ask-vault-password cloud_privateapp_install.yml

# only install/update private apps without removing anything
ansible-playbook -i inventory --ask-vault-password cloud_privateapp_install.yml --skip-tags splunk_cloud_remove_apps
```

```yaml
splunk_cloud_private_app_folder: "./apps/"
```

### install/update/remove a splunkbase app

```bash
ansible-playbook -i inventory --ask-vault-password cloud_splunkbaseapp_install.yml

# only install/update splunkbase apps without removing anything
ansible-playbook -i inventory --ask-vault-password cloud_splunkbaseapp_install.yml --skip-tags splunkbase_remove_app
```

```yaml
splunkbase_username: my-splunk-account-username # please use ansible-vault
splunkbase_password: my-secret-splunk-password # please use ansible-vault
splunkbase_app_list:
 - id: 2968 # SA-cim_vladiator
   version: "1.8.2"
   license: "https://www.apache.org/licenses/LICENSE-2.0.html"
```

### configure splunk cloud ingress rules

```bash
ansible-playbook -i inventory --ask-vault-password cloud_ingress_allow.yml

# only add ingress rules without removing anything
ansible-playbook -i inventory --ask-vault-password cloud_ingress_allow.yml --skip-tags splunk_cloud_ingress_remove
```

```yaml
splunk_cloud_ingress_allow_list:
  search-ui:
    - "1.2.3.4/32"
  search-api:
    - "4.3.2.1/32"
  idm-api:
    - "3.2.1.4/32"
  s2s:
    - "2.1.4.3/32"
  hec:
    - "1.3.4.2/32" 
```

### configure splunk cloud egress rules

```bash
ansible-playbook -i inventory --ask-vault-password cloud_egress_allow.yml

# only add egress rules without removing anything
ansible-playbook -i inventory --ask-vault-password cloud_egress_allow.yml --skip-tags splunk_cloud_egress_remove
```

```yaml
splunk_cloud_egress_allow_list:
  - port: 8089
    subnets:
      - "1.1.1.1/32" # some public endpoint you need to access from splunk cloud
      - "8.8.8.8/32" # some public endpoint you need to access from splunk cloud
```

## Contributing

We welcome any contributions.
If you want to contribute to this project, please read the [contributing guide](CONTRIBUTING.md).

## Code of Conduct

Please read our [Code of Conduct](CODE_OF_CONDUCT.md) as it is our base for interaction.

## License

This project is licensed under the [MIT LICENSE](LICENSE).

## Provider Information

Please visit <https://www.mercedes-benz-techinnovation.com/en/imprint/> for information on the provider.

Notice: Before you use the program in productive use, please take all necessary precautions,
e.g. testing and verifying the program with regard to your specific use.
The program was tested solely for our own use cases, which might differ from yours.

## Legal

* Ansible is a registered trademark of Red Hat, Inc. in the United States and other countries.
* Splunk, Splunk>, and Turn Data Into Doing are trademarks or registered trademarks of Splunk Inc. in the United States and other countries. All other brand names, product names, or trademarks belong to their respective owners. © 2023 Splunk Inc. All rights reserved.
