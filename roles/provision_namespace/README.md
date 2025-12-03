# provision_namespace

An Ansible role for provisioning namespaces in OpenShift clusters. This role supports multiple clusters (endor and coruscant) and provides comprehensive validation, error handling, and idempotent operations.

## Description

This role creates namespaces (projects) in OpenShift clusters using cluster-specific bearer tokens from environment variables. It is designed for use with Ansible Automation Platform (AAP) and includes features for:

- Multi-cluster support (endor and coruscant)
- Comprehensive input validation
- Idempotent namespace creation
- Automatic labeling and annotation
- Verification of namespace creation
- Detailed error handling and reporting

## Requirements

### Ansible Version

- Ansible 2.14 or higher
- Ansible Automation Platform 2.x (recommended)

### Collections

This role requires the `kubernetes.core` collection:

```yaml
# requirements.yml
collections:
  - name: kubernetes.core
    version: ">=2.4.0"
```

Install with:

```bash
ansible-galaxy collection install -r requirements.yml
```

### Python Dependencies

The `kubernetes.core` collection requires:

- Python 3.9+
- kubernetes Python library
- PyYAML
- jsonpatch

### Cluster Access

- Valid bearer tokens for target clusters
- Network access to cluster API endpoints
- Permissions to create namespaces in the target clusters

## Role Variables

### Required Variables

| Variable | Type | Description |
|----------|------|-------------|
| `cluster_name` | string | Target cluster name. Must be `endor` or `coruscant` |
| `namespace_name` | string | Name of the namespace to create. Must follow Kubernetes naming conventions |

### Optional Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `namespace_labels` | dict | `{}` | Custom labels to apply to the namespace |
| `namespace_annotations` | dict | `{}` | Custom annotations to apply to the namespace |
| `provision_namespace_validate_certs` | bool | `false` | Whether to validate SSL certificates |

### Environment Variables

The role expects these environment variables to be set (typically via AAP Custom Credentials):

| Environment Variable | Description |
|---------------------|-------------|
| `ENDOR_API_URL` | API URL for endor cluster (e.g., `https://api.endor.example.com:6443`) |
| `ENDOR_K8S_TOKEN` | Bearer token for endor cluster authentication |
| `CORUSCANT_API_URL` | API URL for coruscant cluster |
| `CORUSCANT_K8S_TOKEN` | Bearer token for coruscant cluster authentication |

### Standard Labels and Annotations

The role automatically applies these standard labels and annotations:

**Standard Labels:**
- `provisioned-by: ansible-aap`

**Standard Annotations:**
- `openshift.io/description: "Namespace provisioned by Ansible Automation Platform"`
- `provisioned-date: <ISO8601 timestamp>`
- `provisioned-cluster: <cluster_name>`

User-provided labels and annotations are merged with these standard values.

## Dependencies

This role has no external role dependencies but requires the `kubernetes.core` collection.

## Example Playbooks

### Basic Usage

```yaml
---
- name: Provision namespace in endor cluster
  hosts: localhost
  gather_facts: true

  roles:
    - role: provision_namespace
      vars:
        cluster_name: endor
        namespace_name: my-app-dev
```

### With Custom Labels and Annotations

```yaml
---
- name: Provision namespace with metadata
  hosts: localhost
  gather_facts: true

  roles:
    - role: provision_namespace
      vars:
        cluster_name: coruscant
        namespace_name: platform-prod
        namespace_labels:
          environment: production
          team: platform
          cost-center: engineering
        namespace_annotations:
          contact: platform-team@example.com
          ticket: JIRA-1234
          description: Production platform namespace
```

### Multiple Namespaces

```yaml
---
- name: Provision multiple namespaces
  hosts: localhost
  gather_facts: true

  tasks:
    - name: Provision development namespace
      ansible.builtin.include_role:
        name: provision_namespace
      vars:
        cluster_name: endor
        namespace_name: myapp-dev
        namespace_labels:
          environment: development

    - name: Provision staging namespace
      ansible.builtin.include_role:
        name: provision_namespace
      vars:
        cluster_name: endor
        namespace_name: myapp-staging
        namespace_labels:
          environment: staging

    - name: Provision production namespace
      ansible.builtin.include_role:
        name: provision_namespace
      vars:
        cluster_name: coruscant
        namespace_name: myapp-prod
        namespace_labels:
          environment: production
```

### With Tags

```yaml
---
- name: Provision namespace with selective execution
  hosts: localhost
  gather_facts: true

  roles:
    - role: provision_namespace
      vars:
        cluster_name: endor
        namespace_name: test-namespace
      tags:
        - provision_namespace
```

Run only validation:
```bash
ansible-playbook playbook.yml --tags validation
```

Skip validation:
```bash
ansible-playbook playbook.yml --skip-tags validation
```

## AAP Integration

### Survey Configuration

Configure an AAP Job Template with a survey:

**Question 1: Cluster Name**
- Variable: `cluster_name`
- Type: Multiple Choice
- Choices: `endor`, `coruscant`
- Required: Yes

**Question 2: Namespace Name**
- Variable: `namespace_name`
- Type: Text
- Required: Yes
- Min Length: 1
- Max Length: 63
- Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$`

**Question 3: Environment Tag (Optional)**
- Variable: `namespace_labels.environment`
- Type: Multiple Choice
- Choices: `development`, `staging`, `production`
- Required: No

**Question 4: Team Tag (Optional)**
- Variable: `namespace_labels.team`
- Type: Text
- Required: No

### Custom Credential Type

Create a custom credential type in AAP:

**Input Configuration:**
```yaml
fields:
  - id: endor_api_url
    type: string
    label: Endor API URL
  - id: endor_k8s_token
    type: string
    label: Endor Bearer Token
    secret: true
  - id: coruscant_api_url
    type: string
    label: Coruscant API URL
  - id: coruscant_k8s_token
    type: string
    label: Coruscant Bearer Token
    secret: true
required:
  - endor_api_url
  - endor_k8s_token
  - coruscant_api_url
  - coruscant_k8s_token
```

**Injector Configuration:**
```yaml
env:
  ENDOR_API_URL: '{{ endor_api_url }}'
  ENDOR_K8S_TOKEN: '{{ endor_k8s_token }}'
  CORUSCANT_API_URL: '{{ coruscant_api_url }}'
  CORUSCANT_K8S_TOKEN: '{{ coruscant_k8s_token }}'
```

## Tags

The role supports the following tags:

| Tag | Description |
|-----|-------------|
| `validation` | Run only validation tasks |
| `configuration` | Run only configuration tasks |
| `provision_namespace` | Run all namespace provisioning tasks |
| `verification` | Run only verification tasks |

## Return Values

The role sets the following statistics that can be queried after execution:

| Statistic | Type | Description |
|-----------|------|-------------|
| `namespace_created` | bool | Whether a new namespace was created (`true`) or already existed (`false`) |
| `namespace_name` | string | Name of the provisioned namespace |
| `target_cluster` | string | Cluster where the namespace was provisioned |
| `namespace_uid` | string | Kubernetes UID of the namespace |
| `operation_timestamp` | string | ISO8601 timestamp of the operation |

Access these in subsequent plays:

```yaml
- name: Display results
  ansible.builtin.debug:
    msg: "Created namespace {{ namespace_name }} with UID {{ namespace_uid }}"
```

## Validation Rules

### Namespace Name Validation

The role enforces Kubernetes naming conventions:

- Must start with a lowercase letter or number
- Can contain only lowercase letters, numbers, and hyphens
- Must end with a lowercase letter or number
- Maximum length: 63 characters
- Pattern: `^[a-z0-9]([-a-z0-9]*[a-z0-9])?$`

**Valid Examples:**
- `my-app`
- `platform-team-dev`
- `app123`
- `123app`

**Invalid Examples:**
- `My-App` (uppercase not allowed)
- `my_app` (underscores not allowed)
- `-myapp` (cannot start with hyphen)
- `myapp-` (cannot end with hyphen)

### Cluster Name Validation

- Must be exactly `endor` or `coruscant`
- Case-sensitive

## Error Handling

The role includes comprehensive error handling:

1. **Pre-flight Validation**: Checks all required variables and their formats before attempting any cluster operations
2. **Credential Validation**: Verifies that environment variables are set and not empty
3. **Namespace Verification**: After creation, verifies the namespace exists and is in Active state
4. **Detailed Error Messages**: Provides specific guidance when operations fail
5. **Graceful Failure**: Uses block/rescue to handle errors and provide clear feedback

## Idempotency

The role is fully idempotent:

- Running the role multiple times with the same parameters will not create duplicate namespaces
- If the namespace already exists, it will be updated with new labels/annotations if provided
- The `namespace_created` statistic indicates whether a new namespace was created or an existing one was updated

## Security Considerations

1. **Token Protection**: Bearer tokens are marked with `no_log: true` to prevent logging
2. **Certificate Validation**: Disabled by default for self-signed certificates; enable in production
3. **Credential Storage**: Use AAP Custom Credentials to securely store tokens
4. **RBAC**: Ensure service accounts have minimal required permissions

## Troubleshooting

### Common Issues

**Issue: "Missing credentials for cluster"**
- Ensure environment variables are set in AAP Custom Credentials
- Verify credential is attached to the Job Template
- Check variable names match exactly (case-sensitive)

**Issue: "Failed to provision namespace"**
- Verify bearer token has not expired
- Confirm service account has `create` permission for namespaces
- Check cluster API URL is correct and accessible
- Verify DNS resolution for cluster API endpoint

**Issue: "Invalid namespace_name"**
- Review Kubernetes naming conventions
- Use only lowercase letters, numbers, and hyphens
- Ensure name starts and ends with alphanumeric character
- Check length is 63 characters or less

**Issue: "Collection kubernetes.core not found"**
- Install the collection: `ansible-galaxy collection install kubernetes.core`
- Verify collection is in the execution environment if using AAP

### Debug Mode

Run with increased verbosity:

```bash
ansible-playbook playbook.yml -vvv
```

## Performance Considerations

- Each namespace creation requires 2 API calls: create and verify
- Role execution time: typically 2-5 seconds per namespace
- Consider batching multiple namespaces in a single playbook for efficiency

## License

MIT

## Author Information

This role was created by the AAP Platform Team.

For questions, issues, or contributions, please contact the platform team or open an issue in the repository.

## Changelog

### Version 1.0.0
- Initial release
- Support for endor and coruscant clusters
- Comprehensive validation and error handling
- AAP integration with Custom Credentials
- Idempotent operations
- Namespace verification
- Standard labels and annotations

## Related Roles

- TBD: Add related roles for namespace configuration, RBAC, quotas, etc.

## Additional Resources

- [Kubernetes Namespaces Documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- [OpenShift Projects Documentation](https://docs.openshift.com/container-platform/latest/applications/projects/working-with-projects.html)
- [Ansible kubernetes.core Collection](https://docs.ansible.com/ansible/latest/collections/kubernetes/core/index.html)
- [Ansible Automation Platform Documentation](https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/)
