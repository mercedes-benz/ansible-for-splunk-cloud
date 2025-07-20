# Uncommitted Changes and Feature Summary

## Latest Changes (July 2025)
- **HEC Token & Index Validation Robustness:**
  - Refactored index pagination and aggregation logic in HEC token role to match Splunk ACS API response (list at root, not 'indexes' key).
  - Fixed allowedIndexes validation: now sets valid_indexes as a fact, ensuring it is available to all tasks and preventing undefined variable errors.
  - Playbook now robustly validates allowedIndexes, is idempotent, and handles errors gracefully.
- **Authentication Improvements:**
  - Ensured splunk_cloud_token is picked up from group_vars, resolving persistent 401 Unauthorized errors.
- **Ansible Output Deprecation Fix:**
  - Updated ansible.cfg to use 'ansible.builtin.default' and 'result_format = yaml', removing deprecated community.general.yaml callback and future-proofing output format.
- **General:**
  - All changes made automatically and robustly, following user preferences for proactive, hands-off fixes.

---

## 1. Master Playbook for Full Deployment
- **Added:** `cloud_full_deploy.yml` — a master playbook to run all main deployment playbooks in sequence for Splunk Cloud.
- **Docs:** Updated `README.md` with instructions for running all playbooks at once using the master playbook.

## 2. Playbook Host Targeting Improvements
- **Changed:** All main playbooks (`cloud_egress_allow.yml`, `cloud_index_config.yml`, `cloud_ingress_allow.yml`, `cloud_privateapp_install.yml`, `cloud_splunkbaseapp_install.yml`) now target:
  - `splunkcloud`
  - Exclude hosts matching `es-*` and `itsi-*`
- **Benefit:** More precise targeting and easier exclusion of non-standard hosts.

## 3. Inventory Update
- **Changed:** `inventory/inventory.yml` now uses `scde-t2eld9k30jvr98hfq` as the host under `splunkcloud` instead of `prod`.

## 4. Dependency Updates
- **Changed:**
  - Downgraded `ansible` from `9.5.1` to `8.7.0`.
  - Upgraded `splunk-appinspect` from `3.0.3` to `3.8.1`.

## 5. Splunk Cloud Index Role Enhancements
- **Changed:** `roles/splunk_cloud_index/defaults/main.yml` — reduced `splunk_cloud_max_index` from 500 to 200.
- **Changed:** Improved logic in `roles/splunk_cloud_index/tasks/check.yml` to better detect when an index needs updating, including `selfStorageBucketPath`.
- **Changed:** `roles/splunk_cloud_index/tasks/create.yml` and `update.yml` now support `selfStorageBucketPath` in index creation/update requests.
- **Changed:** `roles/splunk_cloud_index/tasks/main_loop.yml` and `main.yml` — added debug output and support for `selfStorageBucketPath`.
- **Changed:** Commented out the task that removes indexes not defined in Ansible inventory (in `main.yml`).

## 6. Private App Role Improvements
- **Changed:** `roles/splunk_cloud_private_app/defaults/main.yml` — added more default app names to the exclusion list.
- **Changed:** `roles/splunk_cloud_private_app/tasks/download_loop.yml` — added debug output for app download data and improved header formatting.

## 7. Miscellaneous
- **Changed:** `apps/.gitkeep` file was cleared (no functional impact).

---

## Summary of Features Added
- **Full-stack deployment:** Run all main playbooks in sequence with a single command.
- **Better host targeting:** Exclude unwanted hosts from playbook runs.
- **Index enhancements:** Support for `selfStorageBucketPath` and improved update logic.
- **Private app improvements:** More robust app exclusion and debugging.
- **Dependency management:** Updated Ansible and Splunk AppInspect versions for compatibility and features.
- **HEC and index validation:** Now robust, idempotent, and future-proofed for Splunk ACS API changes.
- **Output handling:** YAML output is now future-proof and deprecation-warning free.

---

**Review and commit these changes as needed.** 