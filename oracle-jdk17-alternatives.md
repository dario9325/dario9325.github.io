---
layout: default
title: Adding Oracle JDK17 in Debian Alternative system
permalink: /oracle-jdk17-alternatives/
---
## Adding Oracle JDK17 LTS in Debian Alternatives system

### Philosophical perspective

> The Debian alternatives system creates a way for several programs that fulfill 
> the same or similar functions to be listed as alternative implementations that 
> are installed simultaneously but with one particular implementation designated 
> as the default.

source: [Debian alternatives](https://wiki.debian.org/DebianAlternatives){:target="_blank"}

If you use Debian you should manage your multiple Java installation under this
system.

### List existing Java alternatives

In your Debian machine, you can query the existing configured Java alternatives
with

```bash
stylee@matrix17:~$ sudo update-java-alternatives -l
java-1.11.0-openjdk-amd64      1111       /usr/lib/jvm/java-1.11.0-openjdk-amd64
jdk-11.0.12                    1112       /usr/lib/jvm/jdk-11.0.12
oracle-java6-jdk-amd64         316        /usr/lib/jvm/oracle-java6-jdk-amd64
oracle-java8-jdk-amd64         318        /usr/lib/jvm/oracle-java8-jdk-amd64
stylee@matrix17:~$ 

```

I installed JDK 17 from Oracle's deb package and I have

```bash
stylee@matrix17:~$ dpkg -l |grep jdk
ii  jdk-17                                                      17-ga                                  amd64        Java Platform Standard Edition Development Kit
stylee@matrix17:~$ 

```

but it doesn't appear in my alternatives configuration. Oracle does not support
the configuration for Debian alternatives.

To enable it, we have to follow the follwing steps.

### Create a jinfo file for JDK 17

Following previous [Oracle JDK11](oracle-jdk11-alternatives/) article,
we can easily adapt it to our version (you can download it [here][1]).

```
name=jdk-17
priority=1700
section=main
hl java /usr/lib/jvm/jdk-17/bin/java
hl jexec /usr/lib/jvm/jdk-17/lib/jexec
hl keytool /usr/lib/jvm/jdk-17/bin/keytool
hl rmiregistry /usr/lib/jvm/jdk-17/bin/rmiregistry
jdk jar /usr/lib/jvm/jdk-17/bin/jar
jdk jarsigner /usr/lib/jvm/jdk-17/bin/jarsigner
jdk javac /usr/lib/jvm/jdk-17/bin/javac
jdk javadoc /usr/lib/jvm/jdk-17/bin/javadoc
jdk javap /usr/lib/jvm/jdk-17/bin/javap
jdk jcmd /usr/lib/jvm/jdk-17/bin/jcmd
jdk jconsole /usr/lib/jvm/jdk-17/bin/jconsole
jdk jdb /usr/lib/jvm/jdk-17/bin/jdb
jdk jdeps /usr/lib/jvm/jdk-17/bin/jdeps
jdk jfr /usr/lib/jvm/jdk-17/bin/jfr
jdk jhsdb /usr/lib/jvm/jdk-17/bin/jhsdb
jdk jimage /usr/lib/jvm/jdk-17/bin/jimage
jdk jinfo /usr/lib/jvm/jdk-17/bin/jinfo
jdk jlink /usr/lib/jvm/jdk-17/bin/jlink
jdk jmap /usr/lib/jvm/jdk-17/bin/jmap
jdk jmod /usr/lib/jvm/jdk-17/bin/jmod
jdk jpackage /usr/lib/jvm/jdk-17/bin/jpackage
jdk jps /usr/lib/jvm/jdk-17/bin/jps
jdk jrunscript /usr/lib/jvm/jdk-17/bin/jrunscript
jdk jshell /usr/lib/jvm/jdk-17/bin/jshell
jdk jstack /usr/lib/jvm/jdk-17/bin/jstack
jdk jstat /usr/lib/jvm/jdk-17/bin/jstat
jdk jstatd /usr/lib/jvm/jdk-17/bin/jstatd
jdk serialver /usr/lib/jvm/jdk-17/bin/serialver
```

Put it in the right position, under

```bash
/usr/lib/jvm/.jdk-17.jinfo
```

Right now, `update-alternatives` just detect it

```bash
stylee@matrix17:~$ sudo update-java-alternatives -l
java-1.11.0-openjdk-amd64      1111       /usr/lib/jvm/java-1.11.0-openjdk-amd64
jdk-11.0.12                    1112       /usr/lib/jvm/jdk-11.0.12
jdk-17                         1700       /usr/lib/jvm/jdk-17
oracle-java6-jdk-amd64         316        /usr/lib/jvm/oracle-java6-jdk-amd64
oracle-java8-jdk-amd64         318        /usr/lib/jvm/oracle-java8-jdk-amd64
stylee@matrix17:~$ 

```

But it's not finished, yet: we have to tell `update-alternatives` which binary
to bind.

### Configure JDK 17 binaries to alternatives system

Launch the following commands

```bash
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk-17/bin/java 1700
sudo update-alternatives --install /usr/bin/jexec jexec /usr/lib/jvm/jdk-17/lib/jexec 1700
sudo update-alternatives --install /usr/bin/keytool keytool /usr/lib/jvm/jdk-17/bin/keytool 1700
sudo update-alternatives --install /usr/bin/rmiregistry rmiregistry /usr/lib/jvm/jdk-17/bin/rmiregistry 1700
sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/jdk-17/bin/jar 1700
sudo update-alternatives --install /usr/bin/jarsigner jarsigner /usr/lib/jvm/jdk-17/bin/jarsigner 1700
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk-17/bin/javac 1700
sudo update-alternatives --install /usr/bin/javadoc javadoc /usr/lib/jvm/jdk-17/bin/javadoc 1700
sudo update-alternatives --install /usr/bin/javap javap /usr/lib/jvm/jdk-17/bin/javap 1700
sudo update-alternatives --install /usr/bin/jcmd jcmd /usr/lib/jvm/jdk-17/bin/jcmd 1700
sudo update-alternatives --install /usr/bin/jconsole jconsole /usr/lib/jvm/jdk-17/bin/jconsole 1700
sudo update-alternatives --install /usr/bin/jdb jdb /usr/lib/jvm/jdk-17/bin/jdb 1700
sudo update-alternatives --install /usr/bin/jdeps jdeps /usr/lib/jvm/jdk-17/bin/jdeps 1700
sudo update-alternatives --install /usr/bin/jfr jfr /usr/lib/jvm/jdk-17/bin/jfr 1700
sudo update-alternatives --install /usr/bin/jhsdb jhsdb /usr/lib/jvm/jdk-17/bin/jhsdb 1700
sudo update-alternatives --install /usr/bin/jimage jimage /usr/lib/jvm/jdk-17/bin/jimage 1700
sudo update-alternatives --install /usr/bin/jinfo jinfo /usr/lib/jvm/jdk-17/bin/jinfo 1700
sudo update-alternatives --install /usr/bin/jlink jlink /usr/lib/jvm/jdk-17/bin/jlink 1700
sudo update-alternatives --install /usr/bin/jmap jmap /usr/lib/jvm/jdk-17/bin/jmap 1700
sudo update-alternatives --install /usr/bin/jmod jmod /usr/lib/jvm/jdk-17/bin/jmod 1700
sudo update-alternatives --install /usr/bin/jpackage jpackage /usr/lib/jvm/jdk-17/bin/jpackage 1700
sudo update-alternatives --install /usr/bin/jps jps /usr/lib/jvm/jdk-17/bin/jps 1700
sudo update-alternatives --install /usr/bin/jrunscript jrunscript /usr/lib/jvm/jdk-17/bin/jrunscript 1700
sudo update-alternatives --install /usr/bin/jshell jshell /usr/lib/jvm/jdk-17/bin/jshell 1700
sudo update-alternatives --install /usr/bin/jstack jstack /usr/lib/jvm/jdk-17/bin/jstack 1700
sudo update-alternatives --install /usr/bin/jstat jstat /usr/lib/jvm/jdk-17/bin/jstat 1700
sudo update-alternatives --install /usr/bin/jstatd jstatd /usr/lib/jvm/jdk-17/bin/jstatd 1700
sudo update-alternatives --install /usr/bin/serialver serialver /usr/lib/jvm/jdk-17/bin/serialver 1700
```
and that's all, it's all configured correctly.

### Use your JDK 17

To use it, run the following

```bash
stylee@matrix17:~$ sudo update-java-alternatives -s jdk-17
update-alternatives: errore: nessuna alternativa per iceweasel-javaplugin.so
update-alternatives: errore: nessuna alternativa per jaotc
update-alternatives: errore: nessuna alternativa per jdeprscan
update-alternatives: errore: nessuna alternativa per jmc
stylee@matrix17:~$ 

```

To prove your java executable is the right one

```bash
stylee@matrix17:~$ java -version
java version "17" 2021-09-14 LTS
Java(TM) SE Runtime Environment (build 17+35-LTS-2724)
Java HotSpot(TM) 64-Bit Server VM (build 17+35-LTS-2724, mixed mode, sharing)
stylee@matrix17:~$ 

```

[1]:{{ site.url }}/download/jdk-17.jinfo