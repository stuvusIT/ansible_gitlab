# gitlab
This role configures and installs a GitLab instance.


## Requirements

This role depends on (postgres)[https://github.com/stuvusIT/postgresql], (redis)[https://github.com/stuvusIT/redis] , (nginx)[https://github.com/stuvusIT/nginx] and Ubuntu

## Role Variables

To enable a variable config for gitlab and since GitLab is also using yaml files you can pretty much write your GitLab config into this role, with the exception that we merge the given config with the default options of GitLab.
To see all GitLab options see the [GitLab example config file](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/config/gitlab.yml.example)

| Variable                                 | Default / Mandatory                                                                                           | Description                                                                                                                         |
|------------------------------------------|---------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| `gitlab_config`                          | :heavy_check_mark:                                                                                            | YAML object containing the GitLab config. This is merged with the default config                                                    |
| `gitlab_database_config`                 | :heavy_check_mark:                                                                                            | YAML object containing the GitLab database config.                                                                                  |
| `gitlab_resque_config`                   | :heavy_check_mark:                                                                                            | YAML object containing the GitLab resque config.                                                                                    |
| `gitlab_secret_config`                   | :heavy_check_mark:                                                                                            | YAML object containing the GitLab secret config.                                                                                    |
| `gitlab_db_name`                         | :heavy_check_mark:                                                                                            | Name of the database to be used. If this databse does not exist it will be created                                                  |
| `gitlab_db_user`                         | :heavy_check_mark:                                                                                            | Name of the databse user. If the user does not exist he will be created                                                             |
| `gitlab_user`                            | `git`                                                                                                         | User under which GitLab should run.                                                                                                 |
| `gitlab_version`                         | `ce`                                                                                                          | Either `ee` or `ce` for Enterprise Edition or Community Edition.                                                                    |
| `gitlab_version_number`                  | :heavy_check_mark:                                                                                            | What version of GitLab should be installed.                                                                                         |
| `gitlab_root_password`                   | :heavy_check_mark:                                                                                            | The password for the Administrator that is created                                                                                  |
| `gitlab_root_mail`                       | :heavy_check_mark:                                                                                            | The email for the Administrator that is createdSets if a project has a wiki section on creation                                     |
| `gitlab_rack_attack_config_path`         | `files/rack_attack.rb`                                                                                        | Path to the rack attack config                                                                                                      |
| `gitlab_postgressql_host`                | :heavy_multiplication_x:                                                                                      | Host where postgresql runs, when set gitlab will not use the socket directive.                                                      |
| `gitlab_unicorn_working_directory`       | `/home/git/gitlab`                                                                                            | Path of the unicorn working directory.                                                                                              |
| `gitlab_unicorn_timeout`                 | `60`                                                                                                          | Timeout in seconds for unicorn.                                                                                                     |
| `gitlab_unicorn_pid`                     | `/home/git/gitlab/tmp/pids/unicorn.pid`                                                                       | Path of pid file for unicorn.                                                                                                       |
| `gitlab_unicorn_stderr_path`             | `/home/git/gitlab/log/unicorn.stderr.log`                                                                     | Stderr log path for unicorn.                                                                                                        |
| `gitlab_unicorn_stdout_path`             | `/home/git/gitlab/log/unicorn.stdout.log`                                                                     | Stdout log path for unicorn.                                                                                                        |
| `gitlab_unicorn_preload_app`             | `true`                                                                                                        | Enabling this preloads an application before forking worker processes.                                                              |
| `gitlab_unicorn_check_client_connection` | `false`                                                                                                       | When enabled, unicorn will check the client connection by writing the beginning of the HTTP headers before calling the application. |
| `gitlab_unicorn_listen`                  | `['"/home/git/gitlab/tmp/sockets/gitlab.socket", :backlog => 1024', '"127.0.0.1:8080", :tcp_nopush => true']` | List of listen directives for unicorn                                                                                               |
| `gitlab_unicorn_worker_processes`        | `3`                                                                                                           | Number of unicorn working processes                                                                                                 |

## Example Playbook

```yml
---
gitlab_db_name: gitlab
gitlab_db_user: gitlab
gitlab_user: git
gitlab_version: ee
gitlab_install_path: /home/git/gitlab
gitlab_version_number: v10.5.4-ee
postgres_initdb: /usr/lib/postgresql/9.5/bin/initdb
postgres_pg_ctl: /usr/lib/postgresql/9.5/bin/pg_ctl
gitlab_root_password: passwords
gitlab_root_mail: root@gitlab.example.com
gitlab_config:
  production: &base
    gitlab:
      host: localhost
      port: 80
      https: false
      email_from: gitlab@example.com
      email_display_name: GitLab
gitlab_database_config:
  production:
    adapter: postgresql
    encoding: unicode
    database: "{{ gitlab_db_name }}"
    pool: 10
    username: "{{ gitlab_db_user }}"
    host: localhost
    socket: /var/run/postgresql
gitlab_resque_config:
  production:
    url: "unix:{{ redis_unixsocket }}"
gitlab_secret_config:
  production:
    db_key_base: mysecretkey
    secret_key_base: mysecretkey2

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

```

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).


## Author Information

 * [Fritz Otlinghaus (Scriptkiddi)](https://github.com/scriptkiddi) _fritz.otlinghaus@stuvus.uni-stuttgart.de_
