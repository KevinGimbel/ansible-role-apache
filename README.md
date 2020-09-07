#  apache

Small and opinionated Apache 2.x role to manage one or more virtual hosts in a Ubuntu environment.

## Features

- Apache 2.x with the following modules:
  - `proxy`
  - `proxy_http`
  - `proxy_http2`
  - `rewrite`
  - `md`
  - `ssl`
- Auto-SSL via Apache and Let's Encrypt, fully managed by Apache `md` module
- Custom, per-host virtual host templates (set `vhost_config_file`)
- fail2ban out of the box with default settings (WIP!)

## Defaults

All settings are _per virtual host!_

- Redirect all HTTP traffic to HTTPS if `use_ssl` is set to true
- Error logs for each vhost
- Default directory index: `DirectoryIndex index.php index.htm index.html`
- Default `AllowOverride All` for document root

## Usage

```
---
- hosts: all
  remote_user: root
  roles:
    - kevingimbel.apache
  vars:
    apache_vhosts:
      - name: "my-site-1"
        fqdn: "awesomewebsite.dev"
        admin_email: "admin@awesomewebsite.dev"
        use_ssl: true

      - name: "kevingimbel"
        fqdn: "next.kevingimbel.de"
        admin_email: "admin@next.kevingimbel.de"
        use_ssl: true
        vhost_config_file: "next.kevingimbel.de.j2"
        document_root: "public"
```

### All virtual host options:

|Option|Description|Default|
|------|-----------|-------|
|`name`|Logical name used in ansible logs, no effect on vhost. |``|
|`fqdn`|Fully Qualified Domain Name, e.g. my-site.com, cool.website, ... |``|
|`admin_email`| E-Mail address used for Let's Encrypt SSL. Required if use_ssl is `true`| `/` |
|`use_ssl`| Set to `true` to create SSL certificates via Apache and Let's Encrypt | `false`|
|`vhost_config_file`| Custom template to use for vhost (see below) | `templates/etc/apache2/sites-available/vhost.conf.j2` |
|`document_root`| Custom document root _inside_ the document root `/var/www/html/{{ item.fqdn }}/` | ``|

```
# Full vhost object
- name: "logialname"
  fqdn: "full-domain.com"
  admin_email: "valid@email.com"
  use_ssl: true
  vhost_config_file: "custom.template.j2"
  document_root: "path/inside/var/www/html/fq
```

### Custom Templates

The custom template is passed the vhost object so whatever is defined on it can be accessed from the template.

```txt
# Custom Template

<VirtualHost *:80>
	ServerName {{ item.fqdn }}
  
  # Custom Headers
  Header set X-Custom-Header "Hello this is a custom header!"
  Header set X-Now-Hiring "We are hiring developers, check out dev.local/jobs"

	DocumentRoot "/var/www/html/{{ item.fqdn }}/public/"
  DirectoryIndex index.php index.htm index.html

  <Directory "/var/www/html/{{ item.fqdn }}/public/">
      AllowOverride All
  </Directory>
</VirtualHost>
```

## Variables

See `defaults/main.yml`

### Disable fail2ban

Set `apache_install_fail2ban` to `false` to disable fail2ban setup.

## License

See LICENSE file.