# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2024-XX-XX

### Added
- Initial release of Cortex Cloud Scan GitHub Action
- Support for Code Security scans (Secrets, IaC, SCA)
- Support for Container Workload Protection scans
- Support for API Security scans
- Automatic cortexcli installation
- Comprehensive input options for all scan types
- Support for SARIF, JSON, and other output formats
- Proxy and CA certificate support
- Examples for all scan types
- Quick start guide
- Complete documentation

### Supported Features
- **Code Security**
  - Secrets scanning with validation
  - Infrastructure-as-Code (IaC) scanning
  - Software Composition Analysis (SCA)
  - Multiple framework support (Terraform, Kubernetes, Docker, etc.)
  - Severity filtering
  - Custom path exclusions
  
- **Container Workload Protection**
  - Image vulnerability scanning
  - Malware detection
  - SBOM generation
  - Docker and OCI archive support
  
- **API Security**
  - OpenAPI/Swagger specification scanning
  - API vulnerability testing
  - Authentication support
  - Configurable concurrency and timeouts

## [Unreleased]

### Planned
- Pre-commit hook examples
- Pre-receive hook examples
- Additional CI/CD platform examples
- Performance optimizations
- Enhanced error reporting

---

[1.0.0]: https://github.com/PaloAltoNetworks/cortex-cloud-scan/releases/tag/v1.0.0
