zabbix_server
=============

Ansible role which helps to install and configure Zabbix server.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Usage
-----

```
- name: Example of default usage with no configuration changes
  hosts: machine1
  roles:
    - postgresql
    - zabbix_server

- name: Example of how to change default DB connection details
  hosts: machine2
  vars:
    # Change engine to MySQL
    zabbix_server_db_engine: mysql
    # The DB is running on a remote machine
    zabbix_server_db_host: 10.0.0.123
    # Admin DB credentials for the zabbix user creation
    zabbix_server_db_login_user: admin
    zabbix_server_db_login_password: s3cr3t
    # Zabbix DB user credentials
    zabbix_server_db_user: zabbix
    zabbix_server_db_password: z4b1x123
  roles:
    - mysql
    - zabbix_server

- name: Example of how to change default Logging configuration
  hosts: machine3
  vars:
    # Increase the default log file size for log rotation to 10MB
    zabbix_server_config_logfilesize: 10
    # Add custom options
    zabbix_server_config__custom:
      # Change the debug level to "Critical information"
      DebugLevel: 1
      # Change frequency of sending unsent alerts from 30 to 5 secs
      SenderFrequency: 5
      # Set timeout to 10 seconds
      Timeout: 10
  roles:
    - zabbix_server

- name: Example of how to write zabbix_server.conf from scratch
  hosts: machine4
  vars:
    zabbix_server_config:
      DBName: zabbix
      DebugLevel: 1
      Timeout: 10
  roles:
    - zabbix_server
```

The default DB engine is set to PostgreSQL but it can be changed to MySQL
(see example above). The change of the engine must be done
before the first run of the role.


Role variables
--------------

```
# Major Zabbix version (used for Zabbix YUM repo)
zabbix_server_major_version: "{{ zabbix_major_version | default(2.4) }}"

# Zabbix YUM repo URL
zabbix_server_yumrepo_url: "{{ zabbix_yumrepo_url | default('http://repo.zabbix.com/zabbix/' ~ zabbix_server_major_version ~ '/rhel/' ~ ansible_distribution_major_version ~ '/$basearch/') }}"

# Additional Zabbix YUM repo params
zabbix_server_yumrepo_params: "{{ zabbix_yumrepo_params | default({}) }}"

# Path to the zabix_server.conf file
zabbix_server_config_file: /etc/zabbix/zabbix_server.conf

# Whether to install EPEL YUM repo
zabbix_server_epel_install: "{{ yumrepo_epel_install | default(true) }}"

# EPEL YUM repo URL
zabbix_server_epel_yumrepo_url: "{{ yumrepo_epel_url | default('https://dl.fedoraproject.org/pub/epel/$releasever/$basearch/') }}"

# EPEL YUM repo GPG key
zabbix_server_epel_yumrepo_gpgkey: "{{ yumrepo_epel_gpgkey | default('https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-$releasever') }}"

# Additional EPEL YUM repo params
zabbix_server_epel_yumrepo_params: "{{ yumrepo_epel_params | default({}) }}"

# DB engine
zabbix_server_db_engine: pgsql

# Package to be installed (exact version can be specified here)
zabbix_server_pkg: zabbix-server-{{ zabbix_server_db_engine }}

# Default PostgreSQL port
zabbix_server_db_port_pgsql_default: 5432

# Default MySQL port
zabbix_server_db_port_mysql_default: 3306

# Initial DB schema directory
zabbix_server_db_schema_dir: /usr/share/doc/zabbix-server-{{ zabbix_server_db_engine }}-*

# User used to create DB and user
zabbix_server_db_login_user: "{{
  'postgres'
    if zabbix_server_db_engine == 'pgsql'
    else
  'root' }}"

# Password used to create DB and user
zabbix_server_db_login_password: "{{
  'postgres'
     if zabbix_server_db_engine == 'pgsql'
     else
  None }}"

# Packages required for the PostgreSQL DB creation
zabbix_server_db_pgsql_pkgs:
    - python-psycopg2
    - postgresql

# Packages required for the MySQL DB creation
zabbix_server_db_mysql_pkgs:
    - MySQL-python
    - mysql


# DB server configuration options
zabbix_server_db_host: localhost
zabbix_server_db_port: "{{
  zabbix_server_db_port_pgsql_default
    if zabbix_server_db_engine == 'pgsql'
    else
  zabbix_server_db_port_mysql_default }}"
zabbix_server_db_name: zabbix
zabbix_server_db_user: zabbix
zabbix_server_db_password: zabbix


# Values of the default options of the Zabbix config
zabbix_server_config_alertscriptspath: /usr/lib/zabbix/alertscripts
zabbix_server_config_externalscripts: /usr/lib/zabbix/externalscripts
zabbix_server_config_pidfile: /var/run/zabbix/zabbix_server.pid
zabbix_server_config_logfile: /var/log/zabbix/zabbix_server.log
zabbix_server_config_logfilesize: 1

# Default options of the Zabbix configuration
zabbix_server_config__default:
  AlertScriptsPath: "{{ zabbix_server_config_alertscriptspath }}"
  ExternalScripts: "{{ zabbix_server_config_externalscripts }}"
  PidFile: "{{ zabbix_server_config_pidfile }}"
  LogFile: "{{ zabbix_server_config_logfile }}"
  LogFileSize: "{{ zabbix_server_config_logfilesize }}"
  DBHost: "{{ zabbix_server_db_host }}"
  DBPort: "{{ zabbix_server_db_port }}"
  DBName: "{{ zabbix_server_db_name }}"
  DBUser: "{{ zabbix_server_db_user }}"
  DBPassword: "{{ zabbix_server_db_password }}"

# Custom options of the Zabbix configuration
zabbix_server_config__custom: {}

# Final Zabbix config
zabbix_server_config: "{{
  zabbix_server_config__default.update(zabbix_server_config__custom) }}{{
  zabbix_server_config__default }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`mysql`](http://github.com/jtyr/ansible-mysql) (optional)
- [`postgresql`](http://github.com/jtyr/ansible-postgresql) (optional)
- [`zabbix_agent`](https://github.com/jtyr/ansible-zabbix_agent) (optional)
- [`zabbix_proxy`](https://github.com/jtyr/ansible-zabbix_proxy) (optional)
- [`zabbix_web`](https://github.com/jtyr/ansible-zabbix_web) (optional)


License
-------

MIT


Author
------

Jiri Tyr
