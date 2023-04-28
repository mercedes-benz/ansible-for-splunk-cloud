# ansible-for-splunk-cloud

## Usage

:warning: Warning: These ansible playbooks will adjust configurations in your Splunk Cloud environment. In this process configurations like apps or indexes will get deleted when not configured in ansible correctly.

### Quickstart

1. Add your Splunk Cloud FQDN to `inventory/inventory.yml`.
2. Generate a token with an user which has the [required capabilities](https://docs.splunk.com/Documentation/SplunkCloud/latest/Config/ACSreqs). The `sc_admin` role has all required capabilities by default.
3. Encrypt the token with `ansible-vault` and place it into `inventory/*/host_vars/vars.yml`.
4. Encrypt your splunkbase username + password and place it into `inventory/group_vars/splunkcloud.yml`
5. Adjust the config files to trigger the workflows/ansible playbooks.

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
