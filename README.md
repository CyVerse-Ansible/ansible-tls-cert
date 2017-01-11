TLS Certificate
=========

Does one of two things depending on configuration:
- Installs your TLS certificate, private key, and CA certificate bundle to target system.
- Deploys a new self-signed certificate + key to target system.

In either case, automatically creates a "full chain" file (containing certificate + CA bundle, suitable for use with Nginx) as well.

Supports Ubuntu 12+ and CentOS 5+, adding support for other distros would be easy.

Caveats / notices:
On CentOS, file permissions for the private key are set to owner root and mode 600, because the enclosing `/etc/pki/tls/private` is permissively visible. (Compare to Ubuntu where the system-wide `/etc/ssl/private` directory already has restricted permissions.) Beyond that, this role does not do anything with file permissions, like configuring additional users/groups which can read the private key. That is left for the deployer to handle in a playbook. This role also does not configure other services that use the installed certificates.

[![Build Status](https://travis-ci.org/CyVerse-Ansible/ansible-tls-cert.svg?branch=master)](https://travis-ci.org/CyVerse-Ansible/ansible-tls-cert)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-tls--cert-blue.svg)](https://galaxy.ansible.com/CyVerse-Ansible/tls-cert/)


Requirements
------------

If using this role to deploy a provided certificate then `openssl` must be available on the deployment host.
If using this role to create and deploy a self-signed certificate then `openssl` must be available on the target host.
(These are likely already true for any modern Linux system.)

Role Variables
--------------

| Variable                | Required | Default             | Choices     | Comments                                   |
|-------------------------|----------|---------------------|-------------|--------------------------------------------|
| TLS_PRIVKEY_SRC_FILE    | no       | false               |             | Path to private key on deployer system     |
| TLS_CERT_SRC_FILE       | no       | false               |             | Path to certificate on deployer system     |
| TLS_CACHAIN_SRC_FILE    | no       | false               |             | Path to CA chain on deployer system        |
| TLS_DEST_BASENAME       | no       | provided cert CN*   |             | Base filename of installed certificate     |
| TLS_CREATE_SELFSIGNED   | no       | false               | true, false | Explicitly creates self-signed certificate |
| TLS_CERT_DEST_DIR       | no       | (distro-specific)   |             | Directory for certificates on target host  |
| TLS_PRIVKEY_DEST_DIR    | no       | (distro-specific)   |             | Directory for private keys on target host  |

If you specify at least a TLS_PRIVKEY_SRC_FILE and a TLS_CERT_SRC_FILE, then the provided files will be installed to the target. (Ansible will look in "files" directory relative to playbook if you specify a bare filename or relative path.) If these variables are not set by the deployer, then the role will create a self-signed certificate instead, with files selfsigned.crt and selfsigned.key. You can explicitly set `TLS_CREATE_SELFSIGNED: true` to override the default behavior and force creation of a self-signed certificate.

`*` If the deployer provides a certificate, then the certificate's indicated CN (domain) will be used as the base name of the files on the target (e.g. `example.com.key`, `example.com.crt`, `example.com.cabundle.crt`, and `example.com.fullchain.crt` for example.com). If this role creates a self-signed certificate, the files will be named with "selfsigned" as the base name. In either case, setting `TLS_DEST_BASENAME` overrides this filename.

`**` By default, certificates and private keys are placed in the distro-specific system-wide default directories (but this can be overridden).


Dependencies
------------

None

Example Playbook
----------------

If you already have a certificate to install:

    - hosts: all
      roles:
         - tls-cert
      vars:
         - TLS_PRIVKEY_SRC_FILE: example.com.key
         - TLS_CERT_SRC_FILE: example.com.crt
         - TLS_CACHAIN_SRC_FILE: example.com.cabundle.crt

If you want to create a self-signed certificate, just call the role with no variables:

    - hosts: all
      roles:
         - tls-cert

License
-------

See license.md

Author Information
------------------

Chris Martin

https://cyverse.org
