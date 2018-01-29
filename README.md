# gitlab

This role configures and installs a GitLab instance.


## Requirements

This role depends on (postgres)[https://github.com/stuvusIT/postgresql], (redis)[https://github.com/stuvusIT/redis] , (nginx)[https://github.com/stuvusIT/nginx] and Ubuntu

## Role Variables

Since Gitlab is higly configureable see the (gitlab example config file)[https://gitlab.com/gitlab-org/gitlab-ce/raw/f441fe7b548fd9cb87eb2f0eadfa88b2e312b692/config/gitlab.yml.example] 
All variable names are gitlab_*. See below for some examples.

| Variable                                              | Default / Mandatory     | Description                                                    |
|-------------------------------------------------------|-------------------------|----------------------------------------------------------------|
| `gitlab_host`                                         | `localhost`             | URL where gitlab is running                                    |
| `gitlab_port`                                         | `80`                    | Default port where gitlab ist listening                        |
| `gitlab_https`                                        | `false`                 | Enable/disable https                                           |
| `gitlab_display_name`                                 | `GitLab`                | Display name                                                   |
| `gitlab_email_reply_to`                               | `noreply@example.com`   | What email address should gitlab use                           |
| `gitlab_email_subject_suffix`                         | ` `                     | Suffix for email subjects                                      |
| `gitlab_default_can_create_group`                     | `true`                  | Enable users to create groups                                  |
| `gitlab_username_changing_enabled`                    | `true`                  | Enable users to change their username                          |
| `gitlab_issue_closing_pattern`                        |                         | What words close a ticket when used in a commit message        |
| `gitlab_default_projects_features_issues`             | `true`                  | Set a if a project has an issue section on creation            |
| `gitlab_default_projects_features_merge_requests`     | `true`                  | Sets if a project has an merge request section on creation     |
| `gitlab_default_projects_features_wiki`               | `true`                  | Sets if a project has a wiki section on creation               |
| `gitlab_default_projects_features_snippets`           | `true`                  | Sets if a project has a snippets section on creation           |
| `gitlab_default_projects_features_builds`             | `true`                  | Sets if a project has a builds section on creation             |
| `gitlab_default_projects_features_container_registry` | `true`                  | Sets if a project has a container registry section on creation |
| `gitlab_webhook_timeout`                              | `10`                    | Seconds till a webhook timesout                                |
| `gitlab_repository_downloads_path`                    | `shared/cache/archive/` | Default paths to search for downloadble files                  |

## Role vars

| Variable                                              | Default / Mandatory     | Description                                                    |
|-------------------------------------------------------|-------------------------|----------------------------------------------------------------|
| `gitlab_rack_attack_config_path`                                         | `files/rack_attack.rb`             | Path to the rack attack config |

## Example Playbook

```yml
---
- hosts: all
  become: true
  vars:
    nginx_access_log: "/var/log/nginx/gitlab_access.log gitlab_access"
    nginx_error_log: "/var/log/nginx/gitlab_error.log"
    nginx_log_format: 
      |
        gitlab_access $remote_addr - $remote_user [$time_local] "$request_method $gitlab_filtered_request_uri $server_protocol" $status $body_bytes_sent "$gitlab_filtered_http_referer" "$http_user_agent"
    upstreams:
        - name: gitlab-workhorse
          path: "unix:/home/git/gitlab/tmp/sockets/gitlab-workhorse.socket fail_timeout=0"
    maps:
          - condition: $http_upgrade $connection_upgrade_gitlab
            content:
            |
              default upgrade;
              ''      close
          - condition: $request_uri $gitlab_temp_request_uri_1
            content:
            |
              default $request_uri;
              ~(?i)^(?<start>.*)(?<temp>[\?&]private[\-_]token)=[^&]*(?<rest>.*)$ "$start$temp=[FILTERED]$rest"
          - condition: $gitlab_temp_request_uri_1 $gitlab_temp_request_uri_2
            content:
            |
              default $gitlab_temp_request_uri_1;
              ~(?i)^(?<start>.*)(?<temp>[\?&]authenticity[\-_]token)=[^&]*(?<rest>.*)$ "$start$temp=[FILTERED]$rest"
          - condition: $gitlab_temp_request_uri_2 $gitlab_filtered_request_uri
            content:
            |
              default $gitlab_temp_request_uri_2;
              ~(?i)^(?<start>.*)(?<temp>[\?&]rss[\-_]token)=[^&]*(?<rest>.*)$ "$start$temp=[FILTERED]$rest"
          - condition: $http_referer $gitlab_filtered_http_referer
            content:
            |
              default $http_referer;
              ~^(?<temp>.*)\? $temp
    served_domains:
      - domains: 
          - gitlab
          - git
        default_server: true
        allowed_ip_ranges:
          - 172.27.10.0/24
        https: false
        enable_http2: true
        configurations:
          - content:
            |
              server_tokens off; ## Don't show the nginx version number, a security best practice
              real_ip_header X-Real-IP; ## X-Real-IP or X-Forwarded-For or proxy_protocol
              real_ip_recursive off;
        locations:
          - condition: /
            content:
            |
              client_max_body_size 0;
              gzip off;

              ## https://github.com/gitlabhq/gitlabhq/issues/694
              ## Some requests take more than 30 seconds.
              proxy_read_timeout      300;
              proxy_connect_timeout   300;
              proxy_redirect          off;

              proxy_http_version 1.1;

              proxy_set_header    Host                $http_host;
              proxy_set_header    X-Real-IP           $remote_addr;
              proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
              proxy_set_header    X-Forwarded-Proto   $scheme;
              proxy_set_header    Upgrade             $http_upgrade;
              proxy_set_header    Connection          $connection_upgrade_gitlab;

              proxy_pass http://gitlab-workhorse;
          - condition: ~ ^/(404|422|500|502|503)\.html$
            content:
            |
              root /home/git/gitlab/public;
              internal;
        
    gitlab_db_user: gitlab
    gitlab_db_name: gitlab
    gitlab_version: ee
    gitlab_install_path: /home/git/gitlab
    gitlab_version_number: v10.1.0-ee
    gitlab_email_from: gitlab@stuvus.uni-stuttgart.de
    postgres_initdb: /usr/lib/postgresql/9.5/bin/initdb
    postgres_pg_ctl: /usr/lib/postgresql/9.5/bin/pg_ctl
    gitlab_root_password: fuckingchangethis
    gitlab_root_mail: fritz.otlinghaus@stuvus.uni-stuttgart.de
    gitlab_secret_key: 99d1b9b8517fd8b8c57d41f9687fa9c1099bf85b07961a0ab7adc2d3a7f10406044172626afb18e3fed169b8eb35307d1377d6d2c7063e5f5d0a3d6615302cd3
    gitlab_ldap_servers:
      main:
        label: 'LDAP'
        host: 'ldaps://ldap01.faveve.uni-stuttgart.de'
        port: 389
        uid: 'sAMAccountName'
        bind_dn: '_the_full_dn_of_the_user_you_will_bind_with'
        password: '_the_password_of_the_bind_user'
        encryption: 'plain'
        verify_certificates: true
        ca_file: ''
        ssl_version: ''
        timeout: 10
        active_directory: false
        allow_username_or_email_login: false
        block_auto_created_users: false
        base: ''
        user_filter: ''
        group_base: ''
        admin_group: ''
        external_groups: []
        sync_ssh_keys: false
        attributes:
          username: ['uid', 'userid', 'sAMAccountName']
          email:    ['mail', 'email', 'userPrincipalName']
          name:       'cn'
          first_name: 'givenName'
          last_name:  'sn'
  roles:
    - redis
    - postgresql
    - gitlab
    - nginx

```

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).


## Author Information

 * [Fritz Otlinghaus (Scriptkiddi)](https://github.com/scriptkiddi) _fritz.otlinghaus@stuvus.uni-stuttgart.de_
