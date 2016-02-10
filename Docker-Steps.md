# Docker Container Creation

## Docker Machines

For my work environment, I already have the default docker-machine which I'll use for running machines I use to run everyday applications. I'll also have a dev docker-machine to do development work and a labs docker-machine for services I am evaluating or learning to use.

- `dev` Development

- `labs` Experimentation

- `default` Production

## Create the docker machine

```bash
docker-machine create --driver virtualbox dev
eval $(docker-machine env dev)
cd $PROJECTS/devops/docker
```

## Build images

Create a hydra-jetty for FEDORA3 image, create another one named hydra-jetty. Also create a hydra-dev image for hydra development.

```bash
cd $PROJECTS/devops/docker/hydra-jetty-fedora3
docker build -t library/hydra-jetty:7.2.0 .
docker build -t library/hydra-jetty .
```

Create Ruby development image
```bash
cd $PROJECTS/devops/docker/ruby-dev
docker build -t library/ruby-dev:2.2.2 .
docker build -t library/ruby-dev .
```

```bash
cd $PROJECTS/devops/docker/hydra-dev
docker build -t hydra-dev:jdk8 .
```

## Startup containers

```bash
cd $PROJECTS/hydra/hydra-jetty
docker run -d --name jetty hydra-jetty:7.2.0
cd $PROJECTS/hydra/cdm/tul_cdm
docker run -itdP -v $(pwd):/app --name hydra-cdm --link jetty hydra-dev:jdk8
```

To set fedora3 work director see: https://wiki.apache.org/solr/SolrConfigXml

```bash
docker run -dP -v $(pwd):/data --name jetty-fedora4 hydra-jetty:latest
```

Configure Hydra on Docker to use the Jetty server for Fedora and Sufia

```bash
rake config:docker
```

To set start the rails app from docker you must bind to the IP address to the exposed IP address.

1. Get the ip address with ifconfig
2. Bind the rails app to the address on eth0.

```bash
export ETH0_IP=`ifconfig eth0 | grep -oP '(?<=inet addr:)[0-9.]*'`
rails server -b $ETH0_IP
```

To log into the hydra-dev container, note the following session and output

```bash
$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                                                                                              NAMES
6a531f760abe        hydra-dev:jdk8         "/usr/sbin/sshd -D"      5 seconds ago       Up 4 seconds        0.0.0.0:32771->22/tcp, 0.0.0.0:32770->3000/tcp, 0.0.0.0:32769->8983/tcp, 0.0.0.0:32768->8986/tcp   hydra-cdm
6cc5d6ea8bff        hydra-jetty:7.2.0      "java -Xmx512m -XX:Ma"   29 seconds ago      Up 29 seconds       8983/tcp, 8986/tcp                                                                                 jetty
$ ssh-docker dev 32771
The authenticity of host '[192.168.99.103]:32771 ([192.168.99.103]:32771)' can't be established.
RSA key fingerprint is SHA256:JGMXNPMS7u2DWL/FPlQXXCMMJVWFLDFO1T9lj0pWxOQ.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.99.103]:32771' (RSA) to the list of known hosts.
root@192.168.99.103's password:<rootpa55>
[root@6a531f760abe ~]#
```

*NOTE:* For the development environment, use the default root password, 'rootpa55'. Remember to change this in a development environment

Configure the Hydra application

```bash
[root@6a531f760abe ~]# cd /app
[root@6a531f760abe app]# bundle install
[root@6a531f760abe app]# rake db:migrate
[root@6a531f760abe app]# rake jetty:start
D, [2016-02-10T15:01:56.085539 #21260] DEBUG -- : Starting jetty with these values:
D, [2016-02-10T15:01:56.085822 #21260] DEBUG -- : jetty_home: /app/jetty
D, [2016-02-10T15:01:56.086409 #21260] DEBUG -- : jetty_command: java -Djetty.port=8986 -Dsolr.solr.home=/app/jetty/solr -XX:MaxPermSize=128m -Xmx256m -jar start.jar
W, [2016-02-10T15:01:56.090419 #21260]  WARN -- : Logging jettywrapper stdout to /app/jetty/jettywrapper.log
D, [2016-02-10T15:01:56.096739 #21260] DEBUG -- : Wrote pid file to /app/tmp/pids/_app_jetty_development.pid with value 21263
W, [2016-02-10T15:02:01.099021 #21260]  WARN -- : Waited 5 seconds for jetty to start, but it is not yet listening on port 8986. Continuing anyway.
I, [2016-02-10T15:02:01.099572 #21260]  INFO -- : Started jetty (5011.0ms)
jetty started at PID 21263
[root@6a531f760abe app]# rails s
=> Booting WEBrick
=> Rails 4.1.6 application starting in development on http://0.0.0.0:3000
=> Run `rails server -h` for more startup options
=> Notice: server is listening on all interfaces (0.0.0.0). Consider using 127.0.0.1 (--binding option)
=> Ctrl-C to shutdown server
[2016-02-10 15:02:43] INFO  WEBrick 1.3.1
[2016-02-10 15:02:43] INFO  ruby 2.2.2 (2015-04-13) [x86_64-linux]
[2016-02-10 15:02:43] INFO  WEBrick::HTTPServer#start: pid=21307 port=3000
```

Visit the development server at `[http://192.168.99.103:32770](http://192.168.99.103:32770)`

You can run jetty locally or connect to the hydra-jetty [TBA]
