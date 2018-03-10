# gitlab
This role configures and installs a GitLab instance.


## Requirements

This role depends on (postgres)[https://github.com/stuvusIT/postgresql], (redis)[https://github.com/stuvusIT/redis] , (nginx)[https://github.com/stuvusIT/nginx] and Ubuntu

## Role Variables

To enable a variable config for gitlab and since GitLab is also using yaml files you can pretty much write your GitLab config into this role, 
with the exception that we merge the given config with the default options of GitLab.
To see all GitLab options see the [GitLab example config file](https://gitlab.com/gitlab-org/gitlab-ce/raw/f441fe7b548fd9cb87eb2f0eadfa88b2e312b692/config/gitlab.yml.example) 

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

```

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).


## Author Information

 * [Fritz Otlinghaus (Scriptkiddi)](https://github.com/scriptkiddi) _fritz.otlinghaus@stuvus.uni-stuttgart.de_
