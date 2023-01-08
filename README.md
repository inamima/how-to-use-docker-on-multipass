# how-to-use-multipass-docker

- [Multipass](https://multipass.run/)
- [cloud-init](https://cloudinit.readthedocs.io/en/latest/)

## Set up

`cloud-init` を使ってインスタンスを立ち上げる。

```shellsession
$ multipass launch -n fakedocker --cloud-init cloud-init.yaml
Launched: fakedocker

$ multipass list
Name                    State             IPv4             Image
fakedocker              Running           192.168.64.3     Ubuntu 22.04 LTS
                                          172.17.0.1
```

shell接続。

```shellsession
$ multipass shell fakedocker
```

cloud-initのログを確認。

```shellsession
$ cat /var/log/cloud-init.log

$ exit
```

ホストOS(mac)のディレクトリをマウント

```shellsession
$ multipass mount /Users fakedocker:/Users
$ multipass mount /private/tmp fakedocker:/tmp
```

インスタンスを削除したいときは

```shellsession
$ multipass stop fakedocker
$ multipass delete fakedocker
$ multipass purge
```

## Docker on multipass

`multipass exec {instance name} -- {command}` の形式でインスタンス上のコマンドを直接実行できる。

例えば [hello-world](https://hub.docker.com/_/hello-world) を実行するには

```shellsession
$ multipass exec fakedocker -- docker run --rm --name hello-world-test hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Dockerfileから実行する場合は、

```shellsession
$ multipass exec fakedocker -- docker build -t ubuntu-nginx docker-nginx/

$ multipass exec fakedocker -- docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
ubuntu-nginx   latest    94fe771887b6   59 seconds ago   225MB
ubuntu         22.04     a8780b506fa4   3 weeks ago      77.8MB

$ multipass exec fakedocker -- docker run --rm --name ubuntu-nginx-test -d -p 8080:80 ubuntu-nginx
ee28bc0199f906cf40d45159c22a9343130b35cc50ccfaacc3b553f809c6176f
```

IPアドレスを確認して `curl` を実行。

```shellsession
$ multipass info fakedocker
Name:           fakedocker
State:          Running
IPv4:           192.168.64.3
                172.17.0.1
Release:        Ubuntu 22.04.1 LTS
...

$ curl 192.168.64.3:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to DISCO MONSTER!</title>
...
```

後始末

```shellsession
$ multipass exec fakedocker -- docker stop ubuntu-nginx-test
# --rm オプションを付けていたら不要
$ multipass exec fakedocker -- docker rm ubuntu-nginx-test
$ multipass exec fakedocker -- docker rmi ubuntu-nginx
```
