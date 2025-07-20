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

### run all playbooks at once (full deployment)

You can run all main playbooks in sequence using the master playbook:

```bash
ansible-playbook -i inventory --ask-vault-password cloud_full_deploy.yml
```

This will execute all the main deployment runbooks in order, including:
- HEC token configuration
- Role and user management
- Splunkbase app installation
- Ingress/egress allowlist configuration
- Index configuration
- Private app installation
- DDSS storage location setup
- Limits.conf configuration
- Maintenance window management

Edit `cloud_full_deploy.yml` to add or remove playbooks as needed.

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
   permissions: # optional
     sharing: "global"
     read: "*"
     write: "sc_admin,power"
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

### Manage Splunk Cloud Roles

Configure roles in `inventory/group_vars/splunkcloud/vars.yml`:

```yaml
splunk_cloud_role_list:
  - name: example_role
    capabilities: [list_all_objects, edit_user]
    state: present
```

Run:

```bash
ansible-playbook -i inventory cloud_role_config.yml
```

### Manage Splunk Cloud Users

Configure users in `inventory/group_vars/splunkcloud/vars.yml`:

```yaml
splunk_cloud_user_list:
  - name: example_user
    roles: [example_role]
    state: present
```

Run:

```bash
ansible-playbook -i inventory cloud_user_config.yml
```

Note: On user creation, a strong 10-digit password is generated and outputted.

### Manage Splunk Cloud HEC Tokens

Configure HEC tokens in `inventory/group_vars/splunkcloud/vars.yml`:

```yaml
splunk_cloud_hec_token_list:
  - name: example_hec_token
    indexes: [main]
    state: present
```

Run:

```bash
ansible-playbook -i inventory cloud_hec_token_config.yml
```

### Manage DDSS Self Storage Locations

Configure DDSS (Dynamic Data Self-Storage) locations in `inventory/group_vars/splunkcloud/ddss_storage.yml`:

```yaml
splunk_cloud_ddss_storage_list:
  - title: "primary-s3-storage"
    bucket_name: "your-deployment-prefix-my-company-ddss-primary"
    folder: "archived-data"
    description: "Primary S3 storage location for DDSS archive data"
  - title: "secondary-gcs-storage"
    bucket_name: "your-deployment-prefix-my-company-ddss-secondary"
    folder: "backup-data"
    description: "Secondary GCS storage location for DDSS backup data"
```

**Important**: The `bucket_name` must include your deployment-specific prefix. Check the error message from your first run to get the exact prefix required (e.g., `scde-xxxxx-yyyyy-`).

Run:

```bash
ansible-playbook -i inventory --ask-vault-password cloud_ddss_storage.yml

# only create/update DDSS storage locations without removing anything
ansible-playbook -i inventory --ask-vault-password cloud_ddss_storage.yml --skip-tags splunk_cloud_remove_ddss_storage
```

### Manage limits.conf Configurations

Configure limits.conf stanza settings in `inventory/group_vars/splunkcloud/limits_conf.yml`:

```yaml
splunk_cloud_limits_conf_stanzas:
  - name: "join"
    settings:
      subsearch_maxout: 100000
      subsearch_maxtime: 60
  - name: "search"
    settings:
      max_searches_per_cpu: 1
      max_realtime_searches: 10
  - name: "subsearch"
    settings:
      maxout: 10000
      maxtime: 60
```

Run:

```bash
ansible-playbook -i inventory --ask-vault-password cloud_limits_conf.yml
```

**Note**: Changing limits.conf settings can affect the performance of your Splunk Cloud Platform deployment. All settings are automatically reloaded after updates.

### Manage Maintenance Windows and Change Freeze Requests

Configure change freeze requests in `inventory/group_vars/splunkcloud/maintenance_window.yml`:

```yaml
splunk_cloud_change_freeze_requests:
  - name: "holiday-freeze-2024"
    start_time: "2025-12-20T00:00:00Z"
    end_time: "2026-01-05T23:59:59Z"
    reason: "Holiday change freeze period"
    applies_to: "Splunk Initiated Changes Only"
    state: present
  - name: "emergency-freeze"
    start_time: "2025-08-15T00:00:00Z"
    end_time: "2025-08-20T23:59:59Z"
    reason: "Emergency maintenance freeze"
    applies_to: "Splunk Initiated Changes Only"
    state: present
  - name: "old-freeze-to-remove"
    state: absent
```

Run:

```bash
ansible-playbook -i inventory --ask-vault-password cloud_maintenance_window.yml
```

**Important Notes**:
- Change freeze requests have a maximum of three months per calendar year
- Emergency Maintenance and Security & Platform Maintenance are not eligible for change freezes
- The `applies_to` field typically should be set to "Splunk Initiated Changes Only"
- Use `state: absent` to remove existing change freeze requests

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
