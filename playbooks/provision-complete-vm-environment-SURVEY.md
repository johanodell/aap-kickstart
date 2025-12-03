# AAP Survey Configuration for Complete VM Environment Provisioning

This document describes the AAP survey configuration for the `provision-complete-vm-environment.yaml` playbook.

## Playbook Overview

This playbook provisions a complete VM environment including:
1. **Namespace** - Creates namespace if it doesn't exist
2. **NetworkAttachmentDefinition** - Creates NAD if it doesn't exist
3. **VirtualMachine** - Creates the VM with specified configuration

## Survey Questions

Configure the following survey questions in your AAP Job Template:

### 1. Target Cluster

| Field | Value |
|-------|-------|
| **Variable** | `cluster_name` |
| **Question** | Target Cluster |
| **Answer Type** | Multiple Choice (single select) |
| **Required** | Yes |
| **Choices** | endor<br>coruscant |
| **Default** | endor |

### 2. Namespace

| Field | Value |
|-------|-------|
| **Variable** | `namespace_name` |
| **Question** | Namespace |
| **Answer Type** | Text |
| **Required** | Yes |
| **Minimum Length** | 1 |
| **Maximum Length** | 63 |
| **Default** | |

### 3. VLAN ID

| Field | Value |
|-------|-------|
| **Variable** | `vlan_id` |
| **Question** | VLAN ID |
| **Answer Type** | Integer |
| **Required** | Yes |
| **Minimum** | 1 |
| **Maximum** | 4094 |
| **Default** | 100 |

**Note:** The NAD name will be automatically generated as `vlan{VLAN_ID}-net` (e.g., `vlan100-net`)

### 4. VM Name

| Field | Value |
|-------|-------|
| **Variable** | `vm_name` |
| **Question** | VM Name |
| **Answer Type** | Text |
| **Required** | Yes |
| **Minimum Length** | 1 |
| **Maximum Length** | 63 |
| **Default** | |

### 5. Instance Type

| Field | Value |
|-------|-------|
| **Variable** | `instance_type` |
| **Question** | Instance Type |
| **Answer Type** | Multiple Choice (single select) |
| **Required** | Yes |
| **Choices** | u1.small (1 vCPU, 2Gi RAM)<br>u1.medium (1 vCPU, 4Gi RAM)<br>u1.large (2 vCPU, 8Gi RAM)<br>u1.xlarge (4 vCPU, 16Gi RAM)<br>u1.2xlarge (8 vCPU, 32Gi RAM)<br>u1.4xlarge (16 vCPU, 64Gi RAM)<br>u1.8xlarge (32 vCPU, 128Gi RAM) |
| **Default** | u1.medium |

### 6. OS Preference

| Field | Value |
|-------|-------|
| **Variable** | `preference` |
| **Question** | Operating System Preference |
| **Answer Type** | Multiple Choice (single select) |
| **Required** | Yes |
| **Choices** | rhel.10<br>rhel.9<br>rhel.8<br>centos.stream9<br>centos.stream8<br>fedora |
| **Default** | rhel.9 |

**Note:** The boot volume DataSource is automatically selected based on the OS preference:
- `rhel.10` → DataSource: `rhel10`
- `rhel.9` → DataSource: `rhel9`
- `rhel.8` → DataSource: `rhel8`
- `centos.stream9` → DataSource: `centos-stream9`
- `centos.stream8` → DataSource: `centos-stream8`
- `fedora` → DataSource: `fedora`

### 7. Boot Volume DataSource Override (Optional)

| Field | Value |
|-------|-------|
| **Variable** | `boot_volume_source_override` |
| **Question** | Boot Volume DataSource Override (leave empty for automatic) |
| **Answer Type** | Text |
| **Required** | No |
| **Default** | (empty) |

**Use Case:** Override the automatic DataSource mapping for custom or non-standard DataSource names.

**Example:** If you have a custom DataSource named `rhel9-custom` or `rhel9-golden-image`, enter it here.

### 8. Root Disk Size

| Field | Value |
|-------|-------|
| **Variable** | `root_disk_size` |
| **Question** | Root Disk Size |
| **Answer Type** | Text |
| **Required** | Yes |
| **Default** | 30Gi |

**Valid Formats:** Use Kubernetes resource format (e.g., `30Gi`, `50Gi`, `100Gi`)

### 9. Auto-start VM

| Field | Value |
|-------|-------|
| **Variable** | `vm_running` |
| **Question** | Start VM Immediately |
| **Answer Type** | Multiple Choice (single select) |
| **Required** | Yes |
| **Choices** | true<br>false |
| **Default** | true |

### 10. SSH Public Keys (Optional)

| Field | Value |
|-------|-------|
| **Variable** | `ssh_authorized_keys` |
| **Question** | SSH Public Keys (one per line) |
| **Answer Type** | Textarea |
| **Required** | No |
| **Default** | |

**Example:**
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC... user@host
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... user@host
```

## Credential Configuration

The playbook requires cluster-specific credentials to be attached to the job template:

1. Attach the appropriate cluster credential type (e.g., `Endor OpenShift` or `Coruscant OpenShift`)
2. The credential should inject the following environment variables:
   - `{CLUSTER}_API_URL` - API endpoint
   - `{CLUSTER}_K8S_TOKEN` - Bearer token
   - `{CLUSTER}_VERIFY_SSL` - SSL verification (optional)
   - `{CLUSTER}_CA_CERT` - CA certificate (optional)

## Example Survey Values

### Example 1: Development VM (Automatic DataSource Mapping)

```yaml
cluster_name: endor
namespace_name: dev-vms
vlan_id: 100
vm_name: dev-workstation-01
instance_type: u1.medium
preference: rhel.9
boot_volume_source_override: ""  # Empty - uses automatic mapping (rhel9)
root_disk_size: 30Gi
vm_running: true
ssh_authorized_keys: |
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC... developer@laptop
```

### Example 2: Production Database Server (Automatic DataSource Mapping)

```yaml
cluster_name: coruscant
namespace_name: production-databases
vlan_id: 200
vm_name: postgres-primary
instance_type: u1.2xlarge
preference: rhel.9
boot_volume_source_override: ""  # Empty - uses automatic mapping (rhel9)
root_disk_size: 100Gi
vm_running: true
ssh_authorized_keys: |
  ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... dba@server
```

### Example 3: Test VM with Custom DataSource

```yaml
cluster_name: endor
namespace_name: testing
vlan_id: 300
vm_name: test-vm-01
instance_type: u1.small
preference: rhel.9
boot_volume_source_override: "rhel9-golden-image"  # Uses custom DataSource
root_disk_size: 20Gi
vm_running: false
ssh_authorized_keys: ""
```

## Playbook Behavior

### Automatic DataSource Mapping

The playbook automatically maps OS preferences to DataSource names, eliminating the need to specify both separately:

| OS Preference | DataSource Name |
|---------------|-----------------|
| `rhel.10` | `rhel10` |
| `rhel.9` | `rhel9` |
| `rhel.8` | `rhel8` |
| `centos.stream9` | `centos-stream9` |
| `centos.stream8` | `centos-stream8` |
| `fedora` | `fedora` |

**Override:** If you need to use a custom DataSource name (e.g., `rhel9-golden-image`), use the `boot_volume_source_override` survey field.

### Idempotency

The playbook is designed to be idempotent:

1. **Namespace**: If the namespace already exists, it will be reused (not recreated)
2. **NAD**: If the NetworkAttachmentDefinition already exists, it will be reused (not recreated)
3. **VM**: The VM will be created or updated based on the redhat.openshift.k8s module behavior

### NAD Naming Convention

The NetworkAttachmentDefinition is automatically named based on the VLAN ID:
- VLAN 100 → `vlan100-net`
- VLAN 200 → `vlan200-net`
- VLAN 1234 → `vlan1234-net`

This ensures consistent naming and makes it easy to identify which VLAN a network is associated with.

### Resource Creation Order

The playbook creates resources in the following order:

1. **Validate** all required variables
2. **Check** if namespace exists
3. **Create namespace** if needed
4. **Check** if NAD exists
5. **Create NAD** if needed
6. **Create VirtualMachine**
7. **Display** access information

## Accessing the Provisioned VM

After successful provisioning, the playbook will display instructions for accessing the VM:

### Console Access
```bash
virtctl console <vm-name> -n <namespace>
```

### Get VM IP Address
```bash
oc get vmi <vm-name> -n <namespace> -o jsonpath='{.status.interfaces[0].ipAddress}'
```

### SSH Access (if keys configured)
```bash
ssh cloud-user@<vm-ip-address>
```

### Check VM Status
```bash
oc get vm <vm-name> -n <namespace>
```

## Troubleshooting

### Common Issues

**Issue:** Namespace creation fails
- **Solution:** Check that the bearer token has permissions to create namespaces

**Issue:** NAD creation fails
- **Solution:** Verify the namespace exists and the token has permission to create NetworkAttachmentDefinitions

**Issue:** DataSource not found
- **Solution:**
  - Check available DataSources: `oc get datasources -n openshift-virtualization-os-images`
  - Verify the DataSource name matches exactly
  - Ensure the DataSource exists before running the playbook

**Issue:** VM fails to start
- **Solution:**
  - Check VM events: `oc describe vm <vm-name> -n <namespace>`
  - Verify the instance type exists: `oc get instancetype`
  - Check DataVolume status: `oc get dv -n <namespace>`

## Best Practices

1. **Use consistent naming conventions** for namespaces and VMs
2. **Document VLAN assignments** to avoid conflicts
3. **Test in development** before creating production VMs
4. **Use appropriate instance types** based on workload requirements
5. **Configure SSH keys** for secure remote access
6. **Set vm_running: false** for VMs that need manual configuration before starting
