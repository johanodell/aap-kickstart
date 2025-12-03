# Changelog

All notable changes to the provision_namespace role will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-12-02

### Added
- Initial release of provision_namespace role
- Support for provisioning namespaces in endor and coruscant OpenShift clusters
- Comprehensive input validation for cluster names and namespace names
- Kubernetes naming convention validation
- Credential validation from environment variables
- Automatic standard labels and annotations
- Custom labels and annotations support
- Idempotent namespace creation using kubernetes.core.k8s module
- Namespace creation verification
- Detailed error handling with block/rescue
- Operation statistics reporting via set_stats
- Tag support for selective task execution
- AAP Custom Credentials integration documentation
- Comprehensive README with usage examples
- Example playbooks for various use cases
- Test playbooks for validation and functionality
- Ansible lint and YAML lint configurations
- Support for RHEL 8, 9, 10, CentOS Stream, and Fedora

### Features
- Multi-cluster support (endor/coruscant)
- FQCN (Fully Qualified Collection Names) for all modules
- Secure token handling with no_log
- Comprehensive role documentation
- Galaxy-compatible role structure
- Production-ready validation rules

### Documentation
- Detailed README.md with examples
- AAP integration guide
- Custom Credential Type configuration
- Survey configuration examples
- Troubleshooting guide
- Variable documentation
- Return values documentation

### Testing
- Basic test playbook
- Validation test suite
- Example playbooks for common patterns
- Loop usage examples

## [Unreleased]

### Planned
- Support for additional clusters beyond endor and coruscant
- NetworkPolicy creation option
- ResourceQuota configuration
- LimitRange configuration
- ServiceAccount creation
- RBAC RoleBinding creation
- Integration tests with molecule
- CI/CD pipeline examples
- Monitoring and alerting integration
- Webhook notification support
- Audit logging enhancements
