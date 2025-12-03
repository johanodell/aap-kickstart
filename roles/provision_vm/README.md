# Ansible Role: provision_vm

Provisions VirtualMachines in OpenShift Virtualization using modern instancetype/preference patterns.

## Description

This role creates VirtualMachines in OpenShift/Kubernetes clusters using OpenShift Virtualization (KubeVirt). It follows modern best practices including:

- U-series instancetype pattern for standardized VM sizing
- DataVolume with sourceRef to DataSource for boot volumes
- Bridge networking to NetworkAttachmentDefinitions (no pod network)
- Cloud-init support for VM customization
- Comprehensive validation of all prerequisites

## Requirements

- Ansible 2.15 or higher
- `kubernetes.core` collection (>= 3.0.0)
- OpenShift cluster with OpenShift Virtualization (KubeVirt) installed
- Valid bearer token with permissions to create VirtualMachines
- Existing DataSource for boot volume
- Existing NetworkAttachmentDefinition for VM networking

## Role Variables

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `cluster_name` | Target cluster (endor or coruscant) | `endor` |
| `namespace_name` | Namespace where VM will be created | `production-vms` |
| `vm_name` | Name of the VirtualMachine | `web-server-01` |
| `instance_type` | U-series instance type | `U1.medium` |
| `preference` | OS preference | `rhel.9` |
| `boot_volume_source` | DataSource name for boot volume | `rhel9` |
| `network_nad_name` | NetworkAttachmentDefinition name | `vlan100-net` |
| `root_disk_size` | Root disk size | `30Gi` |

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `vm_running` | `true` | Whether VM starts running immediately |
| `cloud_init_user_data` | `""` | Custom cloud-init user data |
| `ssh_authorized_keys` | `[]` | SSH public keys for cloud-init |
| `vm_labels` | `{}` | Additional labels for the VM |
| `vm_annotations` | `{}` | Additional annotations for the VM |

### Available Instance Types

| Instance Type | vCPUs | Memory |
|--------------|-------|--------|
| `U1.medium` | 1 | 4Gi |
| `U1.large` | 2 | 8Gi |
| `U1.xlarge` | 4 | 16Gi |
| `U1.2xlarge` | 8 | 32Gi |
| `U1.4xlarge` | 16 | 64Gi |
| `U1.8xlarge` | 32 | 128Gi |

### Available OS Preferences

- `rhel.9`
- `rhel.8`
- `centos.stream9`
- `centos.stream8`
- `fedora`

### Environment Variables Required

The role expects cluster-specific credentials to be injected via environment variables:

For `endor` cluster:
- `ENDOR_API_URL` - OpenShift API endpoint
- `ENDOR_K8S_TOKEN` - Bearer token
- `ENDOR_VERIFY_SSL` - SSL verification (optional, default: true)
- `ENDOR_CA_CERT` - CA certificate (optional)

For `coruscant` cluster:
- `CORUSCANT_API_URL` - OpenShift API endpoint
- `CORUSCANT_K8S_TOKEN` - Bearer token
- `CORUSCANT_VERIFY_SSL` - SSL verification (optional, default: true)
- `CORUSCANT_CA_CERT` - CA certificate (optional)

## Dependencies

- `kubernetes.core` collection

## Example Playbook

### Basic VM Provisioning

```yaml
---
- name: Provision RHEL 9 VM
  hosts: localhost
  gather_facts: false
  roles:
    - role: provision_vm
      vars:
        cluster_name: endor
        namespace_name: production-vms
        vm_name: web-server-01
        instance_type: U1.medium
        preference: rhel.9
        boot_volume_source: rhel9
        network_nad_name: vlan100-bridge
        root_disk_size: 30Gi
```

### VM with SSH Keys

```yaml
---
- name: Provision VM with SSH access
  hosts: localhost
  gather_facts: false
  roles:
    - role: provision_vm
      vars:
        cluster_name: coruscant
        namespace_name: dev-vms
        vm_name: dev-workstation
        instance_type: U1.large
        preference: rhel.9
        boot_volume_source: rhel9
        network_nad_name: dev-network
        root_disk_size: 50Gi
        ssh_authorized_keys:
          - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC..."
```

### VM with Custom Cloud-Init

```yaml
---
- name: Provision VM with custom cloud-init
  hosts: localhost
  gather_facts: false
  roles:
    - role: provision_vm
      vars:
        cluster_name: endor
        namespace_name: production-vms
        vm_name: database-server
        instance_type: U1.2xlarge
        preference: rhel.9
        boot_volume_source: rhel9
        network_nad_name: database-vlan
        root_disk_size: 100Gi
        cloud_init_user_data: |
          #cloud-config
          user: admin
          password: SecurePassword123!
          chpasswd:
            expire: false
          packages:
            - postgresql
            - postgresql-server
          runcmd:
            - postgresql-setup --initdb
            - systemctl enable postgresql
            - systemctl start postgresql
```

### VM That Doesn't Auto-Start

```yaml
---
- name: Provision VM in stopped state
  hosts: localhost
  gather_facts: false
  roles:
    - role: provision_vm
      vars:
        cluster_name: endor
        namespace_name: testing
        vm_name: test-vm
        instance_type: U1.medium
        preference: rhel.9
        boot_volume_source: rhel9
        network_nad_name: test-network
        root_disk_size: 30Gi
        vm_running: false
```

### VM with Custom Labels and Annotations

```yaml
---
- name: Provision VM with metadata
  hosts: localhost
  gather_facts: false
  roles:
    - role: provision_vm
      vars:
        cluster_name: endor
        namespace_name: production-vms
        vm_name: app-server-01
        instance_type: U1.xlarge
        preference: rhel.9
        boot_volume_source: rhel9
        network_nad_name: app-network
        root_disk_size: 50Gi
        vm_labels:
          app: web-application
          tier: frontend
          environment: production
        vm_annotations:
          description: "Production web application server"
          owner: "platform-team@example.com"
          cost-center: "engineering"
```

## AAP Integration

This role is designed to work seamlessly with Ansible Automation Platform job templates and surveys.

### AAP Survey Configuration

Create a survey with the following fields:

| Variable | Question | Type | Required | Choices |
|----------|----------|------|----------|---------|
| `cluster_name` | Target Cluster | Multiple Choice | Yes | endor, coruscant |
| `namespace_name` | Namespace | Text | Yes | |
| `vm_name` | VM Name | Text | Yes | |
| `instance_type` | Instance Type | Multiple Choice | Yes | U1.medium, U1.large, U1.xlarge, U1.2xlarge, U1.4xlarge, U1.8xlarge |
| `preference` | OS Preference | Multiple Choice | Yes | rhel.9, rhel.8, centos.stream9, fedora |
| `boot_volume_source` | Boot Volume DataSource | Text | Yes | |
| `network_nad_name` | Network NAD Name | Text | Yes | |
| `root_disk_size` | Root Disk Size (e.g., 30Gi) | Text | Yes | |
| `vm_running` | Start VM Immediately | Multiple Choice | Yes | true, false |
| `ssh_authorized_keys` | SSH Public Keys (one per line) | Textarea | No | |

### AAP Credential Configuration

1. Create custom credential types for each cluster (see main README.md section 4)
2. Create credentials using these types with your bearer tokens
3. Attach credentials to the job template
4. The role will automatically use the injected environment variables

## Prerequisites Validation

The role validates the following before creating a VM:

1. All required variables are provided
2. Cluster name is valid (endor or coruscant)
3. Instance type is valid U-series type
4. OS preference is valid
5. Cluster credentials are available
6. Target namespace exists
7. DataSource for boot volume exists
8. NetworkAttachmentDefinition exists

If any validation fails, the role will fail with a clear error message.

## VM Access

After the VM is created and running, you can access it using:

### Console Access

```bash
$ virtctl console <vm-name> -n <namespace>
```

### Get VM IP Address

```bash
$ oc get vmi <vm-name> -n <namespace> -o jsonpath='{.status.interfaces[0].ipAddress}'
```

### SSH Access (if SSH keys configured)

```bash
$ ssh cloud-user@<vm-ip-address>
```

## Networking Architecture

This role provisions VMs with **bridge-only networking** - no pod network interface is created. The VM has a single network interface connected to the specified NetworkAttachmentDefinition.

This architecture is ideal for:
- VMs that need direct VLAN access
- VMs that require specific IP addressing from external DHCP
- Production workloads with network isolation requirements

## Boot Volume Pattern

The role uses the modern **DataVolume with sourceRef** pattern:

```yaml
dataVolumeTemplates:
  - metadata:
      name: vm-rootdisk
    spec:
      sourceRef:
        kind: DataSource
        name: <boot_volume_source>
      storage:
        resources:
          requests:
            storage: <root_disk_size>
```

This approach provides:
- Fast cloning from DataSource
- Automatic storage provisioning
- Consistent boot volume management

## License

Apache-2.0

## Author Information

Created by Johan Odell for the AAP Kickstart project.
