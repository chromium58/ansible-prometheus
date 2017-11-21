
williamyeh.prometheus for Ansible Galaxy
============



## Summary

Role name in Ansible Galaxy: **[chromium58.prometheus](https://galaxy.ansible.com/chromium58/prometheus/)**

This Ansible role has the following features for [Prometheus](http://prometheus.io/):

 - Install specific versions of [Prometheus server](https://github.com/prometheus/prometheus), [Node exporter](https://github.com/prometheus/node_exporter), [Alertmanager](https://github.com/prometheus/alertmanager), [Redis exporter](https://github.com/oliver006/redis_exporter), [Postgres exporter](https://github.com/oliver006/https://github.com/wrouesnel/postgres_exporter), [Mysqld exporter](https://github.com/prometheus/mysqld_exporter). Or simply add your own exporter in dictionary and this role will install it for you.
 - Handlers for restart/reload/stop events;
 - Bare bone configuration (*real* configuration should be left to user's template files; see **Usage** section below).




## Role Variables


### Mandatory variables

The components to be installed(i'm setting it in inventory):

```yaml
# Supported components:
#
#   [Server components]
#     - "prometheus"
#     - "alertmanager"
#
#   [Exporter components]
#     - "node_exporter"
#
prometheus_components:
  - prometheus
  - blackbox_exporter
  - alertmanager
prometheus_exporters:
  - node_exporter
  - postgres_exporter
```



### Optional variables: general settings


User-configurable defaults:

```yaml
# user and group
prometheus_user:   prometheus
prometheus_group:  prometheus


# directory for executable files
prometheus_install_path:   /opt/prometheus

# directory for configuration files
prometheus_config_path:    /etc/prometheus

# directory for logs
prometheus_log_path:       /var/log/prometheus

# directory for PID files
prometheus_pid_path:       /var/run/prometheus



# directory for temporary files
prometheus_download_path:  /tmp


# version of helper utility "gosu"
gosu_version:  "1.10"
```


### About systemd


If the Linux distributions are equipped with systemd, this role will use this mechanism accordingly.



### Optional variables: Prometheus server

User-configurable defaults:

```yaml


# directory for rule files
prometheus_rule_path:  {{ prometheus_config_path }}/rules

# directory for file_sd files
prometheus_file_sd_config_path:  {{ prometheus_config_path }}/tgroups

# directory for runtime database
prometheus_db_path:   /var/lib/prometheus
```




User-installable configuration file (see [doc](http://prometheus.io/docs/operating/configuration/) for details):


```yaml
# main conf template relative to `playbook_dir`;
# to be installed to "{{ prometheus_config_path }}/prometheus.yml"
prometheus_conf_main
```


User-installable rule files (see [doc](http://prometheus.io/docs/alerting/rules/) for details):


```yaml
# rule files to be installed to "{{ prometheus_rule_path }}" directory;
# dict fields:
#   - key: memo for this rule
#   - value:
#     - src:  file relative to `playbook_dir`
#     - dest: target file relative to `{{ prometheus_rule_path }}`
prometheus_rule_files
```


Alertmanager to be triggered:

```yaml
prometheus_alertmanager_url
```


Additional command-line arguments, if any (use `prometheus --help` to see the full list of arguments):

```yaml
prometheus_opts
```


### Optional variables: Node exporter


User-configurable defaults:

```yaml
# which version?
prometheus_node_exporter_version:  0.13.0
```

Additional command-line arguments, if any (use `node_exporter --help` to see the full list of arguments):

```yaml
prometheus_node_exporter_opts
```


### Optional variables: Alertmanager


User-configurable defaults:

```yaml
# which version?
prometheus_alertmanager_version:  0.5.1

# directory for runtime database (currently for `silences.json`)
prometheus_alertmanager_db_path: /var/lib/alertmanager
```

User-installable alertmanager conf file (see [doc](http://prometheus.io/docs/alerting/alertmanager/) for details):

```yaml
# main conf template relative to `playbook_dir`;
# to be installed to "{{ prometheus_config_path }}/alertmanager.yml"
prometheus_alertmanager_conf
```

Additional command-line arguments, if any (use `alertmanager --help` to see the full list of arguments):

```yaml
prometheus_alertmanager_opts
```

### Optional variables: Blackbox


User-configurable defaults:

```yaml
# which version?
prometheus_blackbox_exporter_version:  0.4.0
```

User-installable blackbox conf file (see [doc](https://github.com/prometheus/blackbox_exporter) for details):

```yaml
# main conf template relative to `playbook_dir`;
# to be installed to "{{ prometheus_config_path }}/blackbox.yml"
prometheus_blackbox_conf
```

Additional command-line arguments, if any (use `blackbox_exporter --help` to see the full list of arguments):

```yaml
prometheus_blackbox_opts
```


### "Whitebox" exporters dictionary
Exporters versions and dictionary

Exporter options:

  version - expoter version to install(Mandatory option)

  url - install url(Mandatory option)

  binary_path - define it in case if exporter binary is unarchived in non-standart path(optional)

  subdir  - if binary is not inside subdirectory(optional)

  options - command line arguments(optional)

  datasource - option that is used in postgres_exporter, will define environment variable DATASOURCE, that contains connection string to database(optional)

  conf - if exporter have it is own config will install it from
   templates/{{exporter_name}}.yml(optional)

  sudo - if set to yes, exporter will run with root privileges(optional)
```yaml
node_exporter_version: '0.15.0'
redis_exporter_version: '0.13'
postgres_exporter_version: '0.3.0'
mysqld_exporter_version: '0.10.0'
nginx_vts_exporter_version: '0.5'
hpraid_exporter_version: '0.0.3'

prometheus_exporters_dict:
  node_exporter:
    version: '{{node_exporter_version}}'
    url: 'https://github.com/prometheus/node_exporter/releases/download/v{{node_exporter_version}}/node_exporter-{{node_exporter_version}}.linux-amd64.tar.gz'

  redis_exporter:
    version: '{{redis_exporter_version}}'
    url: 'https://github.com/oliver006/redis_exporter/releases/download/v{{redis_exporter_version}}/redis_exporter-v{{redis_exporter_version}}.linux-amd64.tar.gz'
    binary_path: "{{ prometheus_install_path }}/redis_exporter-{{redis_exporter_version}}.{{ prometheus_platform_suffix }}/redis_exporter"
    subdir: no
    options: "-redis.alias={{ inventory_hostname }}"

  postgres_exporter:
    version: '{{postgres_exporter_version}}'
    binary_url: 'https://github.com/wrouesnel/postgres_exporter/releases/download/v{{postgres_exporter_version}}/postgres_exporter'
    datasource: "user=postgres host=/var/run/postgresql/ sslmode=disable"

  mysqld_exporter:
    version: '{{mysqld_exporter_version}}'
    url: 'https://github.com/prometheus/mysqld_exporter/releases/download/v{{mysqld_exporter_version}}/mysqld_exporter-{{mysqld_exporter_version}}.linux-amd64.tar.gz'
    options: '-config.my-cnf={{ prometheus_install_path }}/mysqld_exporter-{{mysqld_exporter_version}}.{{ prometheus_platform_suffix }}/my.cnf'

  nginx_vts_exporter:
    version: '{{nginx_vts_exporter_version}}'
    url: 'https://github.com/hnlq715/nginx-vts-exporter/archive/v{{nginx_vts_exporter_version}}.tar.gz'
    binary_path: "{{ prometheus_install_path }}/nginx_vts_exporter-{{nginx_vts_exporter_version}}.{{ prometheus_platform_suffix }}/nginx-vts-exporter-{{nginx_vts_exporter_version}}/bin/nginx-vts-exporter"
    options: "-nginx.scrape_uri=http://example.com/format/json"
    subdir: no

  hpraid_exporter:
    version: '{{hpraid_exporter_version}}'
    url: 'https://github.com/chromium58/hpraid_exporter/releases/download/v{{hpraid_exporter_version}}/hpraid_exporter-{{hpraid_exporter_version}}.linux-amd64.tar.gz'
    options: '-cmd=/sbin/hpssacli'
    sudo: "yes"

  process_exporter:
    version: '{{process_exporter_version}}'
    url: 'https://github.com/ncabatoff/process-exporter/releases/download/v{{process_exporter_version}}/process-exporter-{{process_exporter_version}}.linux-amd64.tar.gz'
    binary_path: "{{ prometheus_install_path }}/process-exporter-{{process_exporter_version}}.{{ prometheus_platform_suffix }}/process-exporter"
    conf: yes
    options: "-config.path /etc/prometheus/process_exporter.yml"
    sudo: "yes"
```



## Usage


### Step 1: add role

Add role name `chromium58.prometheus` to your playbook file.


### Step 2: add variables

Set vars in your playbook file, if necessary.

Simple example:

```yaml
---
# file: simple-playbook.yml

- hosts: all
  become: True
  roles:
    - chromium58.prometheus

  vars:
    prometheus_components: [ "prometheus", "alertmanager" ]

    prometheus_alertmanager_url: "http://localhost:9093/"
```


### Step 3: copy user's config files, if necessary


More practical example:

```yaml
---
# file: complex-playbook.yml

- hosts: all
  become: True
  roles:
    - chromium58.prometheus

  vars:
    prometheus_components:
      - prometheus
      - node_exporter
      - alertmanager

    prometheus_rule_files:
      this_is_rule_1_InstanceDown:
        src:  some/path/basic.rules
        dest: basic.rules

    prometheus_alertmanager_conf: some/path/alertmanager.yml.j2
```


### Step 4: browse the default Prometheus pages

Open the page in your browser:

- Prometheus - `http://HOST:9090` or `http://HOST:9090/consoles/node.html`

- Alertmanager - `http://HOST:9093`


## Dependencies

None.


## Contributors

- [William Yeh](https://github.com/William-Yeh)
- [Robbie Trencheny](https://github.com/robbiet480) - contribute an early version of building binaries from Go source code.
- [Travis Truman](https://github.com/trumant) - contribute an early version of consul_exporter installer; now moved to [William-Yeh.consul_exporter](https://github.com/William-Yeh/ansible-consul-exporter).
- [Musee Ullah](https://github.com/lae)

## License

MIT License. See the [LICENSE file](LICENSE) for details.
