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

## Requirements

- Ansible 2.13.9 or higher
- Debian 12 or Ubuntu 22.04/24.04 LTS
- amd64 or arm64 architecture
- systemd

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

- BIND9 DLZ backend is validated but not fully implemented
- CTDB clustering requires manual recovery lock configuration
- Certificate management for TLS must be handled externally
- Domain member server testing in containers has networking limitations (resolved
  with static IPs)

## Security considerations

- **Always use Ansible Vault** for domain admin passwords in production
- Default passwords include security warnings
- Kerberos configuration uses modern encryption types for AD compatibility
- All password-handling tasks use `no_log: true` to prevent credential exposure

## Support and maintenance

This is a project maintained by [kogito-ops][] on [GitHub][github-org].

[kogito-ops]: https://kogito.network/ "Kogito UG"
[github-org]: https://github.com/kogito-ops "GitHub: kogito-ops organization"
