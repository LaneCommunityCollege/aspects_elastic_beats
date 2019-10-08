# aspects_elastic_beats

Configure Elastic.co's Beats metric and log shippers.

# Requirements

Set `hash_behaviour=merge` in your ansible.cfg file.

Either use `aspects_packages` to install the Elastic.co yum or apt repo, and `aspects_packages_packages` to install the beats you wish to use from said repo's, or install the beats via another method.
## Using `aspects_packages` to install the packages and repos
```yaml
aspects_packages_enabled: True
aspects_packages_add_repo_yum_repos:
  elastic:
    state: "present"
    name: "Elastic"
    description: "Elastic"
    gpgkey: "https://packages.elastic.co/GPG-KEY-elasticsearch"
    baseurl: "https://artifacts.elastic.co/packages/7.x/yum"
    sslverify: "yes"
    repo_gpgcheck: "yes"
    gpgcheck: "yes"
    enabled: "yes"
    sslcacert: "/etc/ssl/certs/ca-bundle.crt"
    metadata_expire: "300"

aspects_packages_add_repo_apt_repos:
  elastic:
    enabled: True
    key_url: "https://artifacts.elastic.co/GPG-KEY-elasticsearch"
    repo: "deb https://artifacts.elastic.co/packages/7.x/apt stable main"

aspects_packages_packages:
  filebeat:
    state: "present"
    Ubuntu:
      1604: "filebeat"
      1804: "filebeat"
    CentOS:
      7: "filebeat"
    OracleLinux:
      7: "filebeat"
    Debian:
      9: "filebeat"
  metricbeat:
    state: "present"
    Ubuntu:
      1604: "metricbeat"
      1804: "metricbeat"
    CentOS:
      7: "metricbeat"
    OracleLinux:
      7: "metricbeat"
    Debian:
      9: "metricbeat"
  journalbeat:
    state: "absent"
    Ubuntu:
      1604: "journalbeat"
      1804: "journalbeat"
    CentOS:
      7: "journalbeat"
    OracleLinux:
      7: "journalbeat"
    Debian:
      9: "journalbeat"
```
# Role Variables
## aspects_elastic_beats_enabled
To enable or disable this role. Set to True to enable. False to disable.

Default is `False`
## aspects_elastic_beats_global_config
A dictionary of yaml data for settings that apply across all beats. For example, the `output.elasticsearch` or `setup.kibana`
sections. 

The template uses the `to_nice_yaml` Jinja filter to insert the straight yaml configuration at the end of each
beat's main configuration file. 

Follow this pattern:

```yaml
aspects_elastic_beats_global_config:
  enabled: <True|False>
  configuration:
    <straight yaml>
```

## aspects_elastic_beats_beat_config
A dictionary of yaml configuration for the main `*beat.yml` configuration files.

The template uses the `to_nice_yaml` Jinja filter to insert the straight yaml configuration into the target file. 

Follow this pattern:

```yaml
aspects_elastic_beats_beat_config:
  <prefix>beat_yml:
    enabled: <True|False>
    target_path: /etc/<prefix>beat/<prefix>beat.yml
    beat_name: <prefix>beat
    configuration:
      <the basic beat, setup, output, config in yaml format>
  <prefix>beat_yml:
    enabled: <True|False>
    target_path: /etc/<prefix>beat/<prefix>beat.yml
    beat_name: <prefix>beat
    configuration:
      <the basic beat, setup, output, config in yaml format>
```

## aspects_elastic_beats_modules
A dictionary of yaml configuration for the main `/etc/*beat/modules.d/<module>.yml` configuration files.

The template uses the `to_nice_yaml` Jinja filter to insert the straight yaml configuration into the target file. 

Follow this pattern:

```yaml
aspects_elastic_beats_modules:
  <module>:
    enabled: <True|False>
    target_path: /etc/<prefix>beat/modules.d/<module>.yml
    beat_name: <prefix>beat
    configuration:
      <the module config in yaml format>
  <module>:
    enabled: <True|False>
    target_path: /etc/<prefix>beat/modules.d/<module>.yml
    beat_name: <prefix>beat
    configuration:
      <the module config in yaml format>
```


# Example Playbook
```yaml
- hosts: servers
  roles:
     - aspects_elastic_beats
  vars:
    aspects_elastic_beats_enabled: True
    aspects_elastic_beats_global_config:
      enabled: True
      configuration:
        setup:
          template:
            settings:
              index:
                number_of_shards: 1
          kibana:
        output:
          elasticsearch:
            hosts: ["localhost:9200"]
            protocol: "http"
            username: "elastic"
            password: "password"      
    aspects_elastic_beats_beat_config:
      filebeat_yml:
        enabled: True
        target_path: /etc/filebeat/filebeat.yml
        beat_name: filebeat
        configuration:
          filebeat:
            inputs:
              - type: log
                enabled: false
                paths:
                  - /var/log/*.log
            config:
              modules:
                path: ${path.config}/modules.d/*.yml
                reload:
                  enabled: true
      aspects_elastic_beats_modules:
        system:
          enabled: True
          target_path: /etc/filebeat/modules.d/system.yml
          beat_name: filebeat
          configuration:
            - module: system
              syslog:
                enabled: true
                var:
                  paths:
                    - /var/log/*
                    - /var/log/mail/*
              auth:
                enabled: false
```
# License
MIT
