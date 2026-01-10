# Ansible role: `Samba` server management

This Ansible role manages Samba servers in three modes: standalone file servers,
Active Directory domain controllers (primary/secondary), and domain member servers.

## Supported operating systems

- Debian
  - 12 (Bookworm)
- Ubuntu
  - 22.04 LTS (Jammy Jellyfish)
  - 24.04 LTS (Noble Numbat)

## Supported architectures

- amd64 (x86_64)
- arm64 (aarch64)

## Installation

```shell
ansible-galaxy install kogito-ops.samba
```

Install required Ansible collections:

```shell
ansible-galaxy collection install -r requirements.yml
```

## Requirements

- Ansible 2.13.9 or higher
- Debian 12 or Ubuntu 22.04/24.04 LTS
- amd64 or arm64 architecture
- systemd

### Ansible collections

- `ansible.posix`
- `community.general`

## Supported features

### Samba server modes

| Mode                      | Supported | Description                                      |
| ------------------------- | --------- | ------------------------------------------------ |
| Standalone File Server    | ✅        | Traditional Samba file server with local users   |
| AD Controller (Primary)   | ✅        | Primary Active Directory domain controller       |
| AD Controller (Secondary) | ✅        | Secondary Active Directory domain controller     |
| Domain Member Server      | ✅        | Server joined to existing AD domain              |

### Feature matrix

| Feature                     | Standalone | AD DC | Member Server |
| --------------------------- | ---------- | ----- | ------------- |
| Local users and groups      | ✅         | ❌    | ❌            |
| Domain users and groups     | ❌         | ✅    | ✅            |
| Domain authentication       | ❌         | ✅    | ✅            |
| File shares                 | ✅         | ✅    | ✅            |
| User profiles               | ✅         | ✅    | ✅            |
| Kerberos authentication     | ❌         | ✅    | ✅            |
| DNS server (internal)       | ❌         | ✅    | ❌            |
| Winbind integration         | ❌         | ❌    | ✅            |
| ID mapping (rid/autorid/ad) | ❌         | ❌    | ✅            |
| TLS/SSL encryption          | ✅         | ✅    | ✅            |
| Clustering (CTDB)           | ✅         | ✅    | ✅            |
| systemd-resolved integration| ❌         | ✅    | ❌            |

### AD DC features

| Feature                     | Supported | Description                                      |
| --------------------------- | --------- | ------------------------------------------------ |
| Password policies           | ✅        | Complexity, history, age, length requirements    |
| Account lockout policies    | ✅        | Threshold, duration, reset timer                 |
| AD Recycle Bin              | ✅        | Deleted object recovery                          |
| RFC2307 schema              | ✅        | Unix UID/GID attributes for domain accounts      |
| SSH key schema              | ✅        | Centralized SSH public key management            |
| Sudo schema                 | ✅        | Centralized sudo policy management               |
| Service accounts            | ✅        | SPN, keytab export, password management          |
| Organizational Units        | ✅        | OU creation, default computer container          |
| Group Policy Objects        | ✅        | GPO creation, file deployment, OU linking        |
| Domain user RFC2307         | ✅        | Auto-assign Unix attributes to domain users      |
| Domain group RFC2307        | ✅        | Auto-assign Unix attributes to domain groups     |

## Configuration

### Basic usage

#### Standalone file server

```yaml
- hosts: fileservers
  roles:
    - kogito-ops.samba
  vars:
    samba_server_role: standalone server
    samba_workgroup: WORKGROUP
    samba_local_users:
      - name: alice
        password: secret123
    samba_shares:
      - name: documents
        path: /srv/samba/documents
        writable: true
        valid_users: alice
```

#### Primary domain controller

```yaml
- hosts: dc1
  roles:
    - kogito-ops.samba
  vars:
    samba_server_role: domain controller
    samba_create_domain_controller: true
    samba_primary_domain: corp.example.com
    samba_ad_info:
      adminpass: StrongAdminP@ssw0rd
      dns_forwarder: 8.8.8.8
```

#### Secondary domain controller

```yaml
- hosts: dc2
  roles:
    - kogito-ops.samba
  vars:
    samba_server_role: domain controller
    samba_create_domain_controller: true
    samba_primary_domain: corp.example.com
    samba_ad_info:
      adminpass: StrongAdminP@ssw0rd
      dns_forwarder: 8.8.8.8
```

#### Domain member server

```yaml
- hosts: member-servers
  roles:
    - kogito-ops.samba
  vars:
    samba_server_role: member server
    samba_join_domain: true
    samba_primary_domain: corp.example.com
    samba_domain_member:
      domain: corp.example.com
      workgroup: CORP
      realm: CORP.EXAMPLE.COM
      domain_admin_user: Administrator
      domain_admin_password: !vault |
        $ANSIBLE_VAULT;1.1;AES256
        66386439653...
      domain_controller: dc1.corp.example.com
      id_mapping:
        backend: rid
        range_min: 10000
        range_max: 999999
    samba_shares:
      - name: shared
        path: /srv/samba/shared
        writable: true
        valid_users: "@Domain Users"
```

### DNS configuration for AD

For domain joins via DNS lookups, create these DNS SRV records:

```text
SRV   _kerberos._tcp.<domain>                0 100 88  dc-hostname.<domain>
SRV   _ldap._tcp.<domain>                    0 100 389 dc-hostname.<domain>
SRV   _kerberos._tcp.dc._msdcs.<domain>      0 100 88  dc-hostname.<domain>
SRV   _ldap._tcp.dc._msdcs.<domain>          0 100 389 dc-hostname.<domain>
```

## Role variables

### Core settings

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `samba_server_role` | `standalone server` | Server mode: `standalone server`, `domain controller`, or `member server` |
| `samba_create_domain_controller` | `false` | Enable AD domain controller setup |
| `samba_join_domain` | `false` | Enable domain join for member servers |
| `samba_primary_domain` | `samba.internal` | AD domain name |
| `samba_workgroup` | `SAMBA` | Workgroup name (standalone mode) |

### Domain member settings

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `samba_domain_member.domain` | `{{ samba_primary_domain }}` | Domain to join |
| `samba_domain_member.workgroup` | `{{ samba_workgroup }}` | NetBIOS workgroup |
| `samba_domain_member.realm` | `{{ samba_primary_domain \| upper }}` | Kerberos realm |
| `samba_domain_member.domain_admin_user` | `Administrator` | Domain admin username |
| `samba_domain_member.domain_admin_password` | `ChangeMe123!` | Domain admin password (use Ansible Vault!) |
| `samba_domain_member.domain_controller` | `{{ samba_primary_domain_controller }}` | Domain controller hostname |
| `samba_domain_member.id_mapping.backend` | `rid` | ID mapping backend: `rid`, `autorid`, `ad`, `tdb` |
| `samba_domain_member.id_mapping.range_min` | `10000` | Minimum UID/GID for domain users |
| `samba_domain_member.id_mapping.range_max` | `999999` | Maximum UID/GID for domain users |

### File paths

| Variable | Default | Description |
| -------- | ------- | ----------- |
| `samba_shares_path` | `/mnt/samba/shares` | Base path for file shares |
| `samba_profiles_path` | `/mnt/samba/profiles` | User profiles location |
| `samba_log_path` | `/var/log/samba` | Log files location |
| `samba_spool_path` | `/var/spool/samba` | Spool directory |

### User and group management

#### Standalone mode

- `samba_local_groups`: List of local Unix groups to create
- `samba_local_users`: List of local users with Samba passwords

#### AD mode

- `samba_domain_groups`: List of domain groups to create
- `samba_domain_users`: List of domain users to create
- `samba_domain_service_accounts`: List of service accounts to create

### AD domain controller settings

#### RFC2307 schema extensions

Unix attributes for domain accounts are enabled by default. To customize:

```yaml
samba_enable_rfc2307: true  # Default: true
samba_rfc2307_id_start: 10000                    # Base ID for default AD groups
samba_rfc2307_domain_user_uid_start: 10100       # Starting UID for domain users
samba_rfc2307_domain_group_gid_start: 10500      # Starting GID for domain groups
samba_rfc2307_service_account_uid_start: 11000   # Starting UID for service accounts
samba_rfc2307_create_unix_admins: true           # Create Unix Admins group
```

#### SSH key schema

SSH public key storage in AD is enabled by default:

```yaml
samba_enable_ssh_schema: true  # Default: true
```

#### Sudo schema

Sudo schema for centralized rules is enabled by default:

```yaml
samba_enable_sudo_schema: true  # Default: true
samba_sudo_create_default_container: true
samba_sudo_container_name: sudoers
```

#### Password policies

```yaml
samba_password_policy:
  complexity: true
  history_length: 24
  min_age: 1
  max_age: 42
  min_length: 14
```

#### Account lockout policies

```yaml
samba_account_policy:
  lockout_duration: 30
  lockout_threshold: 5
  reset_lockout_after: 30
```

#### Service accounts

```yaml
samba_create_domain_service_accounts: true
samba_domain_service_accounts:
  - name: svc_webapp
    password: !vault |
      $ANSIBLE_VAULT;1.1;AES256
      ...
    description: Web Application Service Account
    password_never_expires: true
    spn: HTTP/webapp.example.com
    export_keytab: true
    keytab_path: /etc/krb5.keytab.webapp
    keytab_owner: www-data
    keytab_group: www-data
    keytab_mode: '0600'
```

#### Organizational Units

```yaml
samba_create_domain_organizational_units: true
samba_domain_organizational_units:
  - 'OU=Servers,DC=corp,DC=example,DC=com'
  - 'OU=Workstations,DC=corp,DC=example,DC=com'
samba_default_computer_ou: 'OU=Workstations,DC=corp,DC=example,DC=com'
```

#### Group Policy Objects

```yaml
samba_create_domain_group_policies: true
samba_domain_group_policies:
  - name: Security Settings
    link_to:
      - 'DC=corp,DC=example,DC=com'
    link_option: enforced
    policy_files:
      - type: directory
        path: 'Machine/Preferences/Groups'
      - type: template
        template: 'gpo/security.xml.j2'
        dest: 'Machine/Preferences/Groups/Groups.xml'
        mode: '0644'
```

#### Domain users with RFC2307 attributes

When `samba_enable_rfc2307: true`, users automatically receive Unix attributes:

```yaml
samba_create_domain_users: true
samba_domain_users:
  - name: jdoe
    password: !vault |
      $ANSIBLE_VAULT;1.1;AES256
      ...
    given_name: John
    surname: Doe
    mail_address: jdoe@example.com
    groups:
      - Engineering
      - VPN Users
    # Optional RFC2307 overrides (auto-assigned if omitted):
    # uid: 10150
    # gid: 10500
    # unix_home: /home/jdoe
    # login_shell: /bin/bash
```

#### Domain groups with RFC2307 attributes

```yaml
samba_create_domain_groups: true
samba_domain_groups:
  - name: Engineering
    description: Engineering Team
    members:
      - jdoe
    # Optional: specific GID (auto-assigned if omitted)
    # gid: 10500
```

#### AD Recycle Bin

Enable deleted object recovery:

```yaml
samba_enable_recycle_bin: true
```

### Share configuration

Define shares using the `samba_shares` variable:

```yaml
samba_shares:
  - name: public
    path: /srv/samba/public
    browsable: true
    guest_ok: true
    writable: false
  - name: finance
    path: /srv/samba/finance
    valid_users: "@finance"
    writable: true
    create_mode: '0660'
    directory_mode: '0770'
```

### Advanced features

#### TLS encryption

```yaml
samba_enable_transport_encryption: true
samba_tls_enabled: true
samba_tls_keyfile: /etc/samba/tls/key.pem
samba_tls_certfile: /etc/samba/tls/cert.pem
samba_tls_cafile: /etc/samba/tls/ca.pem
```

#### Clustering with CTDB

```yaml
samba_create_cluster: true
# Note: CTDB recovery lock must be configured manually
```

## Testing

The role includes Molecule tests for all supported configurations:

### Test scenarios

| Scenario | Description |
| -------- | ----------- |
| `default` | Primary + Secondary domain controllers with replication |
| `member-server` | Full DC + member server setup with domain join |
| `member-config` | Member server configuration validation (no domain join) |
| `standalone` | Standalone file server with shares and users |

### Running tests

```bash
# Test default scenario (Primary + Secondary DC)
molecule test

# Test member server domain join
molecule test -s member-server

# Test standalone server
molecule test -s standalone

# Test all scenarios
molecule test --all

# Run only converge for development
molecule converge -s member-server
```

### Advanced testing features

- **Docker networking**: Custom networks with static IPs for AD testing
- **Sequential startup**: Domain controllers start before member servers
- **Service validation**: Service state checking
- **Configuration testing**: Syntax validation and role-specific checks
- **Idempotence testing**: Ensures role runs without changes on repeat execution

## Limitations

### Not yet implemented

- SYSVOL replication between domain controllers
- Domain functional level management
- Domain backup and restore automation
- LAPS (Local Administrator Password Solution) schema
- Fine-grained password policies (PSOs)
- Reverse DNS zone creation
- Sites and Services management
- Comprehensive health monitoring

### Partial implementations

- BIND9 DLZ backend is validated but not fully implemented
- SPN management limited to service account creation (no list/delete)
- DNS forwarder is global (per-DC configuration not supported)

### External requirements

- CTDB clustering requires manual recovery lock configuration
- Certificate management for TLS must be handled externally
- QEMU/KVM recommended for Molecule testing (full VM isolation for AD)

## Security considerations

- **Always use Ansible Vault** for domain admin passwords in production
- All password-handling tasks use `no_log: true` to prevent credential exposure
- Kerberos configuration uses modern encryption types (AES256, AES128)
- LDAP signing enforcement enabled by default
- Modern password hashing (CryptSHA256, CryptSHA512)
- Service accounts created with `/bin/false` shell
- Configurable password complexity and lockout policies

## Support and maintenance

This is a project maintained by [kogito-ops][] on [GitHub][github-org].

[kogito-ops]: https://kogito.network/ "Kogito UG"
[github-org]: https://github.com/kogito-ops "GitHub: kogito-ops organization"
