# Installation

Log in into your instance:

$ ssh -i userX_id_rsa root@IP


Go to [this page](https://gitlab-runner-downloads.s3.amazonaws.com/latest/index.html) and select the 
appropiate runner for our architecture and OS. Since our droplets use 64 bit Ubuntu, 
we will use `deb/gitlab-runner_amd64.deb`

Once we know which package we need, we download it:

```bash
$ curl -LJO "https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb"
```

Install the `.deb` package:

```bash
$ dpkg -i gitlab-runner_amd64.deb
Selecting previously unselected package gitlab-runner.
(Reading database ... 94592 files and directories currently installed.)
Preparing to unpack gitlab-runner_amd64.deb ...
Unpacking gitlab-runner (14.3.0) ...
Setting up gitlab-runner (14.3.0) ...
GitLab Runner: creating gitlab-runner...
Home directory skeleton not used
Runtime platform                                    arch=amd64 os=linux pid=41262 revision=b37d3da9 version=14.3.0
gitlab-runner: the service is not installed
Runtime platform                                    arch=amd64 os=linux pid=41271 revision=b37d3da9 version=14.3.0
gitlab-ci-multi-runner: the service is not installed
Runtime platform                                    arch=amd64 os=linux pid=41296 revision=b37d3da9 version=14.3.0
Runtime platform                                    arch=amd64 os=linux pid=41369 revision=b37d3da9 version=14.3.0
INFO: Docker installation not found, skipping clear-docker-cache
```

Once installed, we can check that the service is up and running using this command:

```bash
$  systemctl status gitlab-runner
● gitlab-runner.service - GitLab Runner
     Loaded: loaded (/etc/systemd/system/gitlab-runner.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-09-29 03:51:33 UTC; 5min ago
   Main PID: 41377 (gitlab-runner)
      Tasks: 8 (limit: 2344)
     Memory: 6.2M
     CGroup: /system.slice/gitlab-runner.service
             └─41377 /usr/bin/gitlab-runner run --working-directory /home/gitlab-runner --config /etc/gitlab-runner/config.toml --service gitlab-runner --user gitlab-runner

Sep 29 03:51:33 droplet-for-user1 systemd[1]: Started GitLab Runner.
Sep 29 03:51:34 droplet-for-user1 gitlab-runner[41377]: Runtime platform                                    arch=amd64 os=linux pid=41377 revision=b37d3da9 version=14.3.0
Sep 29 03:51:34 droplet-for-user1 gitlab-runner[41377]: Starting multi-runner from /etc/gitlab-runner/config.toml...  builds=0
Sep 29 03:51:34 droplet-for-user1 gitlab-runner[41377]: Running in system-mode.
Sep 29 03:51:34 droplet-for-user1 gitlab-runner[41377]:
Sep 29 03:51:34 droplet-for-user1 gitlab-runner[41377]: Configuration loaded                                builds=0
Sep 29 03:51:34 droplet-for-user1 gitlab-runner[41377]: listen_address not defined, metrics & debug endpoints disabled  builds=0
Sep 29 03:51:34 droplet-for-user1 gitlab-runner[41377]: [session_server].listen_address not defined, session endpoints disabled  builds=0
```

# More info

To see how to install it in other Linux distributions or operating systems, please look at
[Install GitLab Runner: methods of all OS](https://docs.gitlab.com/runner/install/linux-manually.html) 
and [Install for Linux (includes ubuntu)](https://docs.gitlab.com/runner/install/)

There are other ways to install the Gitlab Runner, like using 
[a binary file](https://docs.gitlab.com/runner/install/linux-manually.html#using-binary-file) or 
[using Docker](https://docs.gitlab.com/runner/install/docker.html).

