# aspects_elastic_beats

Configure Elastic.co's Beats metric and log shippers.

# Requirements

Set `hash_behaviour=merge` in your ansible.cfg file.

Either use `aspects_packages` to install the Elastic.co yum or apt repo, and `aspects_packages_packages` to install the beats you wish to use from said repo's, or install the beats via another method.

# Role Variables
## aspects_elastic_beats_enabled
To enable or disable this role. Set to True to enable. False to disable.

Default is `False`

## aspects_elastic_beats_general_config
A dictionary of yaml configuration for the main `*beat.yml` configuration files.

The template uses the `to_nice_yaml` Jinja filter to insert the straight yaml configuration into the target file. 

Follow this pattern:

```yaml
aspects_elastic_beats_general_config:
  <prefix>beat_yml:
    target_path: /etc/<prefix>beat/<prefix>beat.yml
    beat_name: <prefix>beat
    configuration:
      <the basic beat, setup, output, config in yaml format>
  <prefix>beat_yml:
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
    target_path: /etc/<prefix>beat/modules.d/<module>.yml
    beat_name: <prefix>beat
    configuration:
      <the module config in yaml format>
  <module>:
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
    aspects_elastic_beats_general_config:
      filebeat_yml:
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
          processors:
            - add_host_metadata: ~
            - add_cloud_metadata: ~
#          - add_docker_metadata: ~
#          - add_kubernetes_metadata: ~
      aspects_elastic_beats_modules:
        system:
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
