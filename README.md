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
# Default usage with no configuration changes
- name: Example 1
  hosts: machine1
  roles:
    - postgresql
    - zabbix_server

# Change default DB connection details
- name: Example 2
  hosts: machine2
  roles:
    - mysql
    - role: zabbix_server
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

# Change default Logging configuration
- name: Example 4
  hosts: machine4
  roles:
    - role: zabbix_server
      vars:
        # Increase the default log file size for log rotation to 10MB
        zabbix_server_logging_filesize: 10
        # Add custom Logging option
        zabbix_server_logging__custom:
          # Change the debug level to "Critical information"
          DebugLevel: 1

# Add custom options
- name: Example 5
  hosts: machine5
  roles:
    - role: zabbix_server
      vars:
        zabbix_server__custom:
          # Change frequency of sending unsent alerts from 30 to 5 secs
          SenderFrequency: 5
          # Set timeout to 10 seconds
          Timeout: 10

# Write zabbix_server.conf from scratch
- name: Example 6
  hosts: machine6
  roles:
    - role: zabbix_server
      vars:
        zabbix_server_config:
          DBName: zabbix
          DebugLevel: 1
          Timeout: 10
```

The default DB engine is set to PostgreSQL but it can be changed to MySQL
(see example above). The change of the engine must be done
before the first run of the role.


Role variables
--------------

```
# Major Zabbix version (used for Zabbix YUM repo)
zabbix_major_version: 2.4

# Zabbix YUM repo URL
zabbix_yumrepo_url: http://repo.zabbix.com/zabbix/{{ zabbix_major_version }}/rhel/{{ ansible_distribution_major_version }}/$basearch/

# Additional Zabbix YUM repo params
zabbix_yumrepo_params: {}

# Path to the zabix_server.conf file
zabbix_server_config_file: /etc/zabbix/zabbix_server.conf

# Whether to install EPEL YUM repo
zabbix_server_epel_install: yes

# EPEL YUM repo URL
zabbix_server_epel_yumrepo_url: "{{ yumrepo_epel_url | default('https://dl.fedoraproject.org/pub/epel/$releasever/$basearch/') }}"

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

# Initial DB schema direcotry
zabbix_server_db_schema_dir: /usr/share/doc/zabbix-server-{{ zabbix_server_db_engine }}-*/create

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


# Custom DB configuration
zabbix_server_db__custom: {}

# Default DB configuration
zabbix_server_db__default:
  "DB setting":
    DBHost: "{{ zabbix_server_db_host }}"
    DBPort: "{{ zabbix_server_db_port }}"
    DBName: "{{ zabbix_server_db_name }}"
    DBUser: "{{ zabbix_server_db_user }}"
    DBPassword: "{{ zabbix_server_db_password }}"

# Final DB configuration
zabbix_server_db: "{{
  zabbix_server_db__default.update(
  zabbix_server_db__custom) }}{{ zabbix_server_db__default }}"


# Logging options
zabbix_server_logging_file: /var/log/zabbix/zabbix_server.log
zabbix_server_logging_filesize: 1

# Custom logging configuration
zabbix_server_logging__custom: {}

# Default logging configuration
zabbix_server_logging__default:
  "Logging":
    LogFile: "{{ zabbix_server_logging_file }}"
    LogFileSize: "{{ zabbix_server_logging_filesize }}"

# Final logging configuration
zabbix_server_logging: "{{
  zabbix_server_logging__default.update(
  zabbix_server_logging__custom) }}{{ zabbix_server_logging__default }}"


# Other configuration options
zabbix_server_pidfile: /var/run/zabbix/zabbix_server.pid
zabbix_server_alert_scripts_path: /usr/lib/zabbix/alertscripts
zabbix_server_externalScripts: /usr/lib/zabbix/externalscripts

# Custom other configuraton
zabbix_server_other__custom: {}

# Default other configuration
zabbix_server_other__default:
  "Other":
    PidFile: "{{ zabbix_server_pidfile }}"
    AlertScriptsPath: "{{ zabbix_server_alert_scripts_path }}"
    ExternalScripts: "{{ zabbix_server_externalScripts }}"

# Final other configuration
zabbix_server_other: "{{
  zabbix_server_other__default.update(
  zabbix_server_other__custom) }}{{ zabbix_server_other__default }}"


# Custom Zabbix configuration
zabbix_server__custom: {}

# Main Zabbix config
zabbix_server__tmp: {}
zabbix_server_config: "{{
  zabbix_server__tmp.update(zabbix_server_db) }}{{
  zabbix_server__tmp.update(zabbix_server_logging) }}{{
  zabbix_server__tmp.update(zabbix_server_other) }}{{
  zabbix_server__tmp.update(zabbix_server__custom) }}{{ zabbix_server__tmp }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`mysql`](http://github.com/jtyr/ansible-mysql) (optional)
- [`postgresql`](http://github.com/jtyr/ansible-postgresql_server) (optional)
- [`zabbix-agent`](https://github.com/jtyr/ansible-zabbix_agent) (optional)
- [`zabbix-web`](https://github.com/jtyr/ansible-zabbix_web) (optional)


License
-------

MIT


Author
------

Jiri Tyr
