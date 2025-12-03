# Ansible Role: provision_nad

Provisions NetworkAttachmentDefinitions (NADs) in OpenShift clusters for VM networking with Multus CNI.

## Description

This role creates NetworkAttachmentDefinitions in OpenShift/Kubernetes clusters to provide additional network interfaces for VirtualMachines. It supports multiple CNI plugins and can auto-generate configurations for common networking scenarios.

## Requirements

- Ansible 2.15 or higher
- `kubernetes.core` collection (>= 3.0.0)
- OpenShift cluster with Multus CNI installed
- Valid bearer token with permissions to create NetworkAttachmentDefinitions

## Role Variables

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `cluster_name` | Target cluster (endor or coruscant) | `endor` |
| `namespace_name` | Namespace where NAD will be created | `vm-network` |
| `nad_name` | Name of the NetworkAttachmentDefinition | `vlan100-net` |

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `nad_type` | `bridge` | CNI plugin type (bridge, macvlan, ipvlan, sriov, etc.) |
| `vlan_id` | `""` | VLAN ID for VLAN tagging |
| `bridge_name` | `""` | Bridge/master interface name |
| `nad_config` | `""` | Custom CNI configuration (JSON string) |
| `nad_labels` | `{}` | Additional labels for the NAD |
| `nad_annotations` | `{}` | Additional annotations for the NAD |

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

### Basic Usage with Auto-Generated Bridge Configuration

```yaml
---
- name: Provision NAD with bridge networking
  hosts: localhost
  gather_facts: false
  roles:
    - role: provision_nad
      vars:
        cluster_name: endor
        namespace_name: vm-network
        nad_name: vlan100-bridge
        nad_type: bridge
        vlan_id: "100"
```

### MACVLAN with Custom Interface

```yaml
---
- name: Provision NAD with MACVLAN
  hosts: localhost
  gather_facts: false
  roles:
    - role: provision_nad
      vars:
        cluster_name: coruscant
        namespace_name: production-vms
        nad_name: macvlan-external
        nad_type: macvlan
        bridge_name: ens192
        vlan_id: "200"
```

### Custom CNI Configuration

```yaml
---
- name: Provision NAD with custom CNI config
  hosts: localhost
  gather_facts: false
  roles:
    - role: provision_nad
      vars:
        cluster_name: endor
        namespace_name: testing
        nad_name: custom-network
        nad_type: bridge
        nad_config: |
          {
            "cniVersion": "0.3.1",
            "name": "custom-network",
            "type": "bridge",
            "bridge": "br-custom",
            "ipam": {
              "type": "static",
              "addresses": [
                {
                  "address": "10.10.10.0/24"
                }
              ]
            }
          }
```

### With Additional Labels and Annotations

```yaml
---
- name: Provision NAD with metadata
  hosts: localhost
  gather_facts: false
  roles:
    - role: provision_nad
      vars:
        cluster_name: endor
        namespace_name: vm-network
        nad_name: labeled-network
        nad_type: bridge
        vlan_id: "300"
        nad_labels:
          environment: production
          team: platform
        nad_annotations:
          description: "Production VM network for platform team"
          contact: "platform-team@example.com"
```

## AAP Integration

This role is designed to work seamlessly with Ansible Automation Platform job templates and surveys.

### AAP Survey Configuration

Create a survey with the following fields:

| Variable | Question | Type | Required |
|----------|----------|------|----------|
| `cluster_name` | Target Cluster | Multiple Choice (endor, coruscant) | Yes |
| `namespace_name` | Namespace | Text | Yes |
| `nad_name` | NAD Name | Text | Yes |
| `nad_type` | CNI Type | Multiple Choice (bridge, macvlan, ipvlan) | Yes |
| `vlan_id` | VLAN ID | Integer | No |
| `bridge_name` | Bridge/Interface Name | Text | No |

### AAP Credential Configuration

1. Create custom credential types for each cluster (see main README.md section 4)
2. Create credentials using these types with your bearer tokens
3. Attach credentials to the job template
4. The role will automatically use the injected environment variables

## Supported CNI Types

The role supports the following CNI plugins:

- **bridge** - Linux bridge networking (auto-config supported)
- **macvlan** - MACVLAN networking (auto-config supported)
- **ipvlan** - IPVLAN networking (auto-config supported)
- **sriov** - SR-IOV networking (custom config required)
- **host-device** - Host device plugin (custom config required)
- **ptp** - Point-to-point networking (custom config required)
- **static** - Static networking (custom config required)
- **tuning** - CNI tuning plugin (custom config required)
- **bandwidth** - Bandwidth limiting (custom config required)
- **firewall** - Firewall plugin (custom config required)

For CNI types without auto-config support, you must provide a custom `nad_config`.

## Usage in VirtualMachines

After creating a NAD with this role, reference it in your VirtualMachine specifications:

```yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: my-vm
  namespace: vm-network
spec:
  running: true
  template:
    spec:
      networks:
      - name: vlan100-net
        multus:
          networkName: vlan100-bridge
      domain:
        devices:
          interfaces:
          - name: vlan100-net
            bridge: {}
```

## License

Apache-2.0

## Author Information

Created by Johan Odell for the AAP Kickstart project.
