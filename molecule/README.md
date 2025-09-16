# Molecule Testing

This directory contains molecule test scenarios for the Samba Ansible role.

## Test Scenarios

- **default**: AD domain controller setup (Ubuntu 22.04)
- **member-config**: Member server configuration test
- **member-server**: Member server joining existing domain
- **standalone**: Standalone server configuration

## Security Configuration

Test passwords are configured using environment variables to avoid hardcoding credentials.

### Using Environment Variables

1. Copy the example environment file:

   ```bash
   cp .env.molecule.example .env.molecule
   ```

2. Set your test passwords in `.env.molecule`:

   ```bash
   export MOLECULE_SAMBA_ADMIN_PASS="YourTestAdminPass123!"
   export MOLECULE_SAMBA_MACHINE_PASS="YourTestMachinePass123!"
   export MOLECULE_DOMAIN_ADMIN_PASS="YourTestDomainAdmin123!"
   export MOLECULE_TEST_USER_PASS="YourTestUserPass123!"
   ```

3. Source the environment file before running tests:

   ```bash
   source .env.molecule
   molecule test
   ```

### Default Test Passwords

If environment variables are not set, the following defaults are used:

- Admin password: `TestAdminPass123!`
- Machine password: `TestMachinePass123!`
- Domain admin password: `TestDomainAdmin123!`
- Test user password: `TestUserPass123!`

**Note**: The `.env.molecule` file is gitignored and should never be committed.

## Running Tests

```bash
# Test default scenario
molecule test

# Test specific scenario
molecule test -s ubuntu

# Development workflow
molecule create
molecule converge
molecule verify
molecule destroy
```

## Network Configuration

Some scenarios use custom networks for proper AD communication:

- member-server: Uses bridge network with static IPs

## Known Issues

- Container systemd workarounds are required for some services
- DNS resolution may need manual configuration in containers
