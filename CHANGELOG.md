# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial release of Ansible role for Samba server management
- Support for multiple Samba server modes:
  - Standalone file server with local users and groups
  - Active Directory Domain Controller (primary and secondary)
  - Domain member server with authentication integration
- Comprehensive Molecule testing framework with QEMU driver
- Support for Ubuntu 22.04 LTS and Debian 12
- Active Directory domain provisioning and management
- Domain user and group creation and management
- NSSwitch and PAM integration for seamless domain authentication
- SSH access control integration with Active Directory groups
- Kerberos configuration for secure AD environments
- AD Recycle Bin support for deleted object recovery
- Configurable password policies for AD domains
- Automatic home directory creation via PAM
- User profile directory management
- Shared directory configuration with flexible permissions
- Production-ready linting and testing standards
- Comprehensive development documentation

### Security
- Passwords and sensitive data protected with no_log directives
- Secure handling of domain administrator credentials
- SSH access controls integrated with AD group membership
- Modern Kerberos encryption for secure authentication

---

## Commit Types

This project uses conventional commits with the following types:

- `feat`: A new feature
- `fix`: A bug fix
- `docs`: Documentation only changes
- `style`: Changes that do not affect the meaning of the code
- `refactor`: A code change that neither fixes a bug nor adds a feature
- `perf`: A code change that improves performance
- `test`: Adding missing tests or correcting existing tests
- `build`: Changes that affect the build system or external dependencies
- `ci`: Changes to CI configuration files and scripts
- `chore`: Other changes that don't modify src or test files