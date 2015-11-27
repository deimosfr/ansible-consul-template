Ansible Consul Template playbook
=====

This role installs and configures Consul-template. It manages HAProxy as well and support systemd boot.

It embends the compiled version of consul template.

More about consul template: https://github.com/hashicorp/consul-template

Requirements
------------

This role requires Ansible 1.6 or higher and platform requirements are listed
in the metadata file.

Role Variables
--------------

The variables that can be passed to this role:

```
consul_template_version: '0.11.0'
consul_template_arch: 'linux_amd64'
consul_template_http_src: "https://github.com/hashicorp/consul-template/releases/download/v{{consul_template_version}}/consul_template_{{consul_template_version}}_{{consul_template_arch}}.zip"

consul_template_bin_path: '/usr/bin'
consul_template_config_dir: '/etc/consul-template'
consul_template_config_file_path: '{{consul_template_config_dir}}/consul-template.conf'
consul_template_manage_service: true

consul_template_consul_addr: '127.0.0.1:8500'
consul_template_token: 'abcd1234'
consul_template_retry: '10s'
consul_template_max_stale: '10m'

consul_template_auth_enable: false
consul_template_username: 'user'
consul_template_password: 'password'

consul_template_ssl_enable: false
consul_template_ssl_verify: 'false'
consul_template_ssl_cert: '/path/to/client/cert.pem'
consul_template_ssl_ca_cert: '/path/to/ca/cert.pem'

consul_template_logfile_path: '/var/log/consul-template.log'
consul_template_syslog_enable: true
consul_template_syslog_facility: 'LOCAL5'
```

Then you can define the haproxy configuration (not mandatory, you can use your own haproxy role):

```
consul_template_install_haproxy: true
consul_template_haproxy_template_dest: '/etc/consul-template/haproxy.cmtpl'
consul_template_haproxy_config:
  - name: 'global'
    config:
      - log /dev/log local0
      - log /dev/log local1 notice
      - chroot /var/lib/haproxy
      - stats socket /run/haproxy/admin.sock mode 660 level admin
      - stats timeout 30s
      - user haproxy
      - group haproxy
      - daemon
      - maxconn 2000
      - ca-base /etc/ssl/certs
      - crt-base /etc/ssl/private
      - ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
      - ssl-default-bind-options no-sslv3
  - name: 'defaults'
    config:
      - log     global
      - mode    http
      - option  httplog
      - option  dontlognull
      - timeout connect 5000
      - timeout client  50000
      - timeout server  50000
      - errorfile 400 /etc/haproxy/errors/400.http
      - errorfile 403 /etc/haproxy/errors/403.http
      - errorfile 408 /etc/haproxy/errors/408.http
      - errorfile 500 /etc/haproxy/errors/500.http
      - errorfile 502 /etc/haproxy/errors/502.http
      - errorfile 503 /etc/haproxy/errors/503.http
      - errorfile 504 /etc/haproxy/errors/504.http
# Add this for consul-template
# Look at consul-template documentation to get more information
#  - name: 'listen web'
#    config:
#      - option forwardfor
#      - bind *:80{{'{{'}}range service "web"{{'}}'}}
#      - server {{'{{'}}.Node{{'}}'}} {{'{{'}}.Address{{'}}'}}:{{'{{'}}.Port{{'}}'}} check{{'{{'}}end{{'}}'}}
```

And to finish the consul-templates can be define like this

```
consul_template_config_file:
  - name: haproxy 
    source: "{{consul_template_haproxy_template_dest}}"
    destination: '/etc/haproxy/haproxy.cfg'
    command: 'service haproxy restart'
```

Examples
========

```
# Roles
- name: consul
  hosts: consul
  user: root
  roles:
    - your-consul-role
    - deimosfr.consul-template
  vars_files:
    - "host_vars/consul.yml"
```

Dependencies
------------

License
-------

GPL2

Author Information
------------------

Pierre Mavro / deimosfr


