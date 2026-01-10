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
- Service account management with security hardening
- NSSwitch and PAM integration for seamless domain authentication
- SSH access control integration with Active Directory groups
- Kerberos configuration for secure AD environments
- AD Recycle Bin support for deleted object recovery
- Configurable password policies and account lockout policies
- Enhanced domain provisioning with security options
- Automatic home directory creation via PAM
- User profile directory management
- Shared directory configuration with flexible permissions
- Production-ready linting and testing standards
- Comprehensive development documentation
- **RFC2307 Schema Extensions** for Unix/Linux attribute support
  - Assigns GID numbers to all 16 default AD groups
  - Special handling for Administrator (no UID) and Domain Admins (no GID)
  - Unix Admins group creation for proper Unix/Linux administration
  - ypServ30 NIS domain configuration
  - Configurable starting UID/GID ranges
  - Idempotent implementation with marker file tracking
- **SSH Key Schema Extensions** for centralized SSH key management
  - OpenSSH LPK (LDAP Public Key) schema implementation
  - sshPublicKey attribute for storing SSH public keys in AD user objects
  - ldapPublicKey auxiliary class added to User objects
  - Schema verification and validation
  - Compatible with standard OpenSSH LPK tools
- **Sudo Schema Extensions** for centralized sudo policy management
  - Complete sudo schema with all standard attributes (sudoUser, sudoHost, sudoCommand, etc.)
  - sudoRole structural class for defining sudo rules in AD
  - Optional sudoers container (OU) creation
  - Schema verification for all attributes
  - Compatible with sudo-ldap client configurations
- Molecule test scenario for RFC2307 schema validation
- **RFC2307 Unix Attributes for Domain Accounts**
  - Domain users receive Unix attributes when RFC2307 is enabled (uidNumber, gidNumber, unixHomeDirectory, loginShell, msSFU30NisDomain, msSFU30Name)
  - Domain groups receive Unix attributes when RFC2307 is enabled (gidNumber, msSFU30NisDomain, msSFU30Name)
  - Service accounts receive Unix attributes when RFC2307 is enabled (with /bin/false shell for security)
  - Configurable UID/GID ranges for account types: `samba_rfc2307_domain_user_uid_start` (default: 10100), `samba_rfc2307_domain_group_gid_start` (default: 10500), `samba_rfc2307_service_account_uid_start` (default: 11000)
  - Domain Users group receives gidNumber 100 (POSIX standard)
- **Domain User Group Membership**
  - Users can be added to multiple groups via the `groups` attribute in `samba_domain_users`
- **Organizational Units Management**
  - OU creation with idempotent existence checks
  - Default computer container redirection via wellKnownObjects attribute
  - Variables: `samba_create_domain_organizational_units`, `samba_domain_organizational_units`, `samba_default_computer_ou`
- **Group Policy Objects Management**
  - GPO creation with GUID tracking
  - Policy file deployment: directories, files, templates, inline content
  - Windows ACL configuration with Domain Computers read access (SDDL)
  - GPO version management and AD synchronization
  - CSE (Client-Side Extension) GUID synchronization for Windows client compatibility
  - GPO linking to OUs with enforce option
  - Variables: `samba_create_domain_group_policies`, `samba_domain_group_policies`
- **Handler for DNS Service Coordination**
  - Proper restart sequence for systemd-resolved and samba-ad-dc when using SAMBA_INTERNAL DNS
- **Handler for Samba Runtime Cleanup**
  - Cleans stale PID files, message queues, sockets, and lock files before service start

### Changed
- **Task Execution Order**: Local resources (groups, users, profiles, shares) now created before AD DC service starts
- **smb.conf Template Improvements**
  - SYSVOL and NETLOGON shares now have proper AD DC configuration (case sensitive = no, smb encrypt = desired, vfs objects)
  - Removed deprecated `reject_md5_clients`/`reject_md5_servers` options from security_options
  - Added explicit `ldap server require strong auth = no` fallback
  - Added documentation comments for SYSVOL/NETLOGON configuration requirements
- **Tag Standardization**
  - Domain account tasks now use `samba_domain_account_management` tag
  - Schema tasks use `samba-ad`, `samba-rfc2307` format
  - Service tasks use `samba-service` tag
- **Defaults Documentation**
  - Expanded examples in `defaults/main.yml` with RFC2307 integration notes
  - Detailed comments for user, group, and service account configuration

### Fixed
- **Idempotence improvements** for reliable repeated execution
  - Domain marker file creation now uses `copy` with `force: false` instead of `state: touch` to prevent false positives
  - Password policy comparison logic enhanced with explicit type conversion to prevent string/integer mismatches
  - Account lockout policy comparison logic enhanced with explicit type conversion
  - Password policy parsing now uses `regex_findall` instead of `regex_search` to avoid Ansible internal `.group()` errors
  - Recycle Bin feature now checks current dsheuristics value before attempting updates
- **Group Creation Error Handling**: Added "already in use" to accepted error messages
- **Service Account Group Membership**: Added `--object-types=user` flag to prevent ambiguous object errors
- **Unix Admins Group Membership**: More robust parsing of ldbsearch output using `select('match')` filter

### Security
- Passwords and sensitive data protected with no_log directives
- Secure handling of domain administrator credentials
- SSH access controls integrated with AD group membership
- Modern Kerberos encryption for secure authentication
- LDAP signing enforcement (require strong auth)
- Modern password hashing (CryptSHA256, CryptSHA512)
- Service account security hardening (false shell, /dev/null home)
- Enhanced provisioning security options
- Ansible Vault support for sensitive configuration data

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