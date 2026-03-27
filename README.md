# role_wordpress

Ansible role to install, configure, and manage WordPress sites. Supports fresh installation via WP-CLI and restoring existing WordPress sites from backup archives (directory, tar.gz, or zip).

## Requirements

- Ansible >= 2.9
- MariaDB/MySQL server available (use `role_mariadb` or external DB)
- Web server (Nginx recommended via `role_nginx`)

## Supported Platforms

- AlmaLinux 9, 10
- RockyLinux 9, 10
- Debian 11 (Bullseye), 12 (Bookworm)
- Ubuntu 22.04 (Jammy), 24.04 (Noble)

## Role Variables

### WordPress Site Configuration

| Variable | Default | Description |
|---|---|---|
| `wordpress_version` | `"6.7.2"` | WordPress version to install |
| `wordpress_domain` | `"example.com"` | Site domain name |
| `wordpress_site_title` | `"My WordPress Site"` | Site title |
| `wordpress_dir` | `/var/www` | Base web directory |
| `wordpress_site_dir` | `/var/www/<domain>` | Site installation path |
| `wordpress_owner` | `www-data` | File owner |
| `wordpress_php_version` | `"8.3"` | PHP version |

### WordPress Admin

| Variable | Default | Description |
|---|---|---|
| `wordpress_admin_user` | `admin` | Admin username |
| `wordpress_admin_password` | `changeme` | Admin password (use vault!) |
| `wordpress_admin_email` | `admin@example.com` | Admin email |

### Database Connection

| Variable | Default | Description |
|---|---|---|
| `wordpress_db_name` | `wordpress` | Database name |
| `wordpress_db_user` | `wordpress` | Database user |
| `wordpress_db_password` | `changeme` | Database password (use vault!) |
| `wordpress_db_host` | `localhost` | Database host |

### Restore from Backup

| Variable | Default | Description |
|---|---|---|
| `wordpress_restore_enabled` | `false` | Enable restore mode |
| `wordpress_restore_files_source` | `""` | Path to backup (directory, .tar.gz, or .zip) |

The restore is idempotent - it only restores if wp-config.php doesn't exist in the target directory.

### SSL Certificates

| Variable | Default | Description |
|---|---|---|
| `wordpress_ssl_enabled` | `false` | Enable SSL certificate deployment |
| `wordpress_ssl_cert_source` | `""` | Path to certificate file |
| `wordpress_ssl_key_source` | `""` | Path to private key |
| `wordpress_ssl_ca_source` | `""` | Path to CA bundle |
| `wordpress_ssl_cert_dir` | `/etc/ssl/wordpress` | Certificate destination |

### Credential Saving

| Variable | Default | Description |
|---|---|---|
| `wordpress_credentials_save_enabled` | `true` | Save credentials to controller |
| `wordpress_credentials_save_path` | `/opt/ansible/credentials/wordpress` | Path on Ansible controller |

## Example Playbook

### Fresh Installation

```yaml
- hosts: wordpress
  become: true
  roles:
    - role: role_base
      tags: base
    - role: role_mariadb
      tags: mariadb
      vars:
        mariadb_root_password: "{{ vault_mariadb_root_password }}"
        mariadb_databases:
          - name: wordpress_db
        mariadb_users:
          - name: wp_user
            password: "{{ vault_wordpress_db_password }}"
            host: localhost
            priv: "wordpress_db.*:ALL"
    - role: role_wordpress
      tags: wordpress
      vars:
        wordpress_domain: "mysite.com"
        wordpress_db_name: wordpress_db
        wordpress_db_user: wp_user
        wordpress_db_password: "{{ vault_wordpress_db_password }}"
        wordpress_admin_password: "{{ vault_wordpress_admin_password }}"
```

### Restore from Backup

```yaml
- hosts: wordpress
  become: true
  roles:
    - role: role_base
      tags: base
    - role: role_mariadb
      tags: mariadb
      vars:
        mariadb_root_password: "{{ vault_mariadb_root_password }}"
        mariadb_databases:
          - name: wordpress_db
        mariadb_restore_enabled: true
        mariadb_restore_source: "backup/dymer.com.tr/db/database-1774563723.tar.gz"
        mariadb_restore_database: wordpress_db
    - role: role_wordpress
      tags: wordpress
      vars:
        wordpress_domain: "dymer.com.tr"
        wordpress_restore_enabled: true
        wordpress_restore_files_source: "backup/dymer.com.tr/wp/"
        wordpress_ssl_enabled: true
        wordpress_ssl_cert_source: "backup/dymer.com.tr/certs/certificate.pem"
        wordpress_ssl_key_source: "backup/dymer.com.tr/certs/private.pem"
        wordpress_ssl_ca_source: "backup/dymer.com.tr/certs/cabundle.pem"
```

## License

GPL-2.0-or-later
