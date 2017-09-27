# TLS Certificate

Installs TLS (SSL) certificates to the target, in one of three ways depending on configuration:
- Installs your provided TLS certificate, private key, and CA certificate bundle to target system.
- Deploys a new self-signed certificate + key to target system.
- Obtains a TLS certificate from [Let's Encrypt](https://letsencrypt.org/) and configures automated certificate renewal via cron

In either case, automatically creates a "full chain" file (containing certificate + CA bundle, suitable for use with Nginx) as well.

Supports Ubuntu LTS 14+ and CentOS 6+, adding support for other distros would be easy.

[![Build Status](https://travis-ci.org/CyVerse-Ansible/ansible-tls-cert.svg?branch=master)](https://travis-ci.org/CyVerse-Ansible/ansible-tls-cert)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-tls--cert-blue.svg)](https://galaxy.ansible.com/CyVerse-Ansible/tls-cert/)

## Role Variables

How to configure? Minimally:
- If you want to deploy a provided certificate, define the `TLS_*_SRC_FILE` vars. (Ansible will look in "files" directory relative to playbook if you specify a bare filename or relative path.)
- If you want to create+deploy a self-signed certificate, don't define any of `TLS_*_SRC_FILE`
- If you want to use Let's Encrypt, set `TLS_LETSENCRYPT: true` and define `TLS_LETSENCRYPT_EMAIL`, consider also defining `TLS_LETSENCRYPT_TLS_SERVICE`
- If not deploying a provided certificate, consider also setting `TLS_COMMON_NAME`

| Variable                      | Required                     | Default             | Choices     | Comments                                                                    |
|-------------------------------|------------------------------|---------------------|-------------|-----------------------------------------------------------------------------|
| TLS_PRIVKEY_SRC_FILE          | no                           |                     |             | Path to private key on deployer system                                      |
| TLS_CERT_SRC_FILE             | no                           |                     |             | Path to certificate on deployer system                                      |
| TLS_CACHAIN_SRC_FILE          | no                           |                     |             | Path to CA chain on deployer system                                         |
| TLS_COMMON_NAME               | no                           | ansible_fqdn        |             | The fully qualified domain for the certficate generated or obtained from LE |
| TLS_DEST_BASENAME             | no                           | provided cert CN*   |             | Base filename of installed certificate (ignored when using LE)              |
| TLS_CERT_DEST_DIR             | no                           | (distro-specific**) |             | Directory for certificates on target host (ignored when using LE)           |
| TLS_PRIVKEY_DEST_DIR          | no                           | (distro-specific**) |             | Directory for private keys on target host (ignored when using LE)           |
| TLS_LETSENCRYPT               | no                           | false               | true, false | Obtain a certificate using Let's Encrypt                                    |
| TLS_LETSENCRYPT_TLS_SERVICE   | no                           |                     |             | Name of service to stop when obtaining certificate and restart upon renewal |
| TLS_LETSENCRYPT_EMAIL         | when TLS_LETSENCRYPT is true |                     |             | Email address to associate with Let's Encrypt certificate                   |

These variables are registered by this role for use in your downstream automations:

| Variable           | Comments                      |
|--------------------|-------------------------------|
| TLS_PRIVKEY_PATH   | Path to private key on target |
| TLS_CERT_PATH      | Path to certificate on target |
| TLS_CACHAIN_PATH   | Path to CA chain on target    |
| TLS_FULLCHAIN_PATH | Path to fullchain on target   |

**`TLS_COMMON_NAME`** is the Common Name (CN) for the generated certificate, and will also be used to set the Subject Alternate Name (SAN), which indicates the host behind the certificate. A valid example would be 'local.atmo.cloud'. The Chrome browser recently made this field mandatory. If you would like to learn about the history read this SO [answer](http://stackoverflow.com/a/14648100/1213041).

`*` If the deployer provides a certificate, then the certificate's indicated CN (domain) will be used as the base name of the files on the target (e.g. `example.com.key`, `example.com.crt`, `example.com.cachain.crt`, and `example.com.fullchain.crt` for example.com). If this role creates a self-signed certificate, the files will be named with "selfsigned" as the base name. In either case, setting `TLS_DEST_BASENAME` overrides this filename.

`**` By default, certificates and private keys are placed in the distro-specific system-wide default directories (but this can be overridden).

## Caveats / Notices
- `openssl` must be available on the _deployment host_ if using this role to deploy a provided certificate, and on the _target host_ if creating+deploying a self-signed certificate. These are likely already true for any modern Linux system.

- On CentOS, file permissions for the private key are set to owner root and mode 600, because the enclosing `/etc/pki/tls/private` is permissively visible. (Compare to Ubuntu where the system-wide `/etc/ssl/private` directory already has restricted permissions.) Beyond that, this role does not do anything with file permissions, like configuring additional users/groups which can read the private key. That is left for the deployer to handle in a playbook. This role also does not configure other services that use the installed certificates.

- When deploying a certificate with Let's Encrypt, certbot will run in standalone mode. Port 443 must be made available for the standalone server to respond to the ACME challenge. You can provide the name of your service listening on port 443 with `TLS_LETSENCRYPT_TLS_SERVICE`, and this service will be temporarily stopped when obtaining/renewing the certificate.

## Dependencies

None

## Example Playbook

If you already have a certificate to install:

    - hosts: all
      roles:
        - tls-cert
      vars:
        - TLS_PRIVKEY_SRC_FILE: 'example.com.key'
        - TLS_CERT_SRC_FILE: 'example.com.crt'
        - TLS_CACHAIN_SRC_FILE: 'example.com.cachain.crt'

If you want to create a self-signed certificate, just call the role with no variables:

    - hosts: all
      roles:
        - tls-cert

If you want to use Let's Encrypt (example Nginx web server):

    - hosts: all
      roles:
        - tls-cert
      vars:
        - TLS_LETSENCRYPT: 'true'
        - TLS_LETSENCRYPT_HTTPS_SERVICE: 'nginx'
        - TLS_LETSENCRYPT_EMAIL: 'dont@spam.me'

## License

See license.md

## Author Information
Chris Martin
Connor Osborn

https://cyverse.org
