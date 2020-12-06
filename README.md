# Welcome to my GitHub pages

Hello! I'm [Dario](about/)

# Adding Oracle JDK11 in Debian Alternatives system

## Philosophical perspective

> The Debian alternatives system creates a way for several programs that fulfill 
> the same or similar functions to be listed as alternative implementations that 
> are installed simultaneously but with one particular implementation designated 
> as the default.

source: [Debian alternatives](https://wiki.debian.org/DebianAlternatives)

If you use Debian you should manage your multiple Java installation under this
system.

## List existing Java alternatives

In your Debian machine, you can query the existing configured Java alternatives
with

```bash
stylee@matrix17:~$ sudo update-java-alternatives -l
java-1.11.0-openjdk-amd64      1111       /usr/lib/jvm/java-1.11.0-openjdk-amd64
oracle-java6-jdk-amd64         316        /usr/lib/jvm/oracle-java6-jdk-amd64
oracle-java8-jdk-amd64         318        /usr/lib/jvm/oracle-java8-jdk-amd64
stylee@matrix17:~$
```

I installed JDK 11.0.6 from Oracle's deb package and I have

```bash
stylee@matrix17:~$ dpkg -l |grep jdk
ii  jdk-11.0.6                                                  11.0.6-1                            amd64        Java Platform Standard Edition Development Kit
stylee@matrix17:~$
```

but it doesn't appear in my alternatives configuration. Oracle does not support
the configuration for Debian alternatives.

To enable it, we have to follow the follwing steps.

## Create a jinfo file for JDK 11

Thanks to [dedeibel](https://gist.github.com/dedeibel/) 
[work](https://gist.github.com/dedeibel/685dc47e6361b341d208b1747cedbc5b), we 
have a working jinfo for a JDK 11.0.2 installation, we can easily adapt it to 
our version.

```
name=jdk-11.0.6
priority=1106
section=main
hl java /usr/lib/jvm/jdk-11.0.6/bin/java
hl jexec /usr/lib/jvm/jdk-11.0.6/lib/jexec
hl keytool /usr/lib/jvm/jdk-11.0.6/bin/keytool
hl pack200 /usr/lib/jvm/jdk-11.0.6/bin/pack200
hl rmid /usr/lib/jvm/jdk-11.0.6/bin/rmid
hl rmiregistry /usr/lib/jvm/jdk-11.0.6/bin/rmiregistry
hl unpack200 /usr/lib/jvm/jdk-11.0.6/bin/unpack200
jdk jar /usr/lib/jvm/jdk-11.0.6/bin/jar
jdk jarsigner /usr/lib/jvm/jdk-11.0.6/bin/jarsigner
jdk javac /usr/lib/jvm/jdk-11.0.6/bin/javac
jdk javadoc /usr/lib/jvm/jdk-11.0.6/bin/javadoc
jdk javap /usr/lib/jvm/jdk-11.0.6/bin/javap
jdk jcmd /usr/lib/jvm/jdk-11.0.6/bin/jcmd
jdk jconsole /usr/lib/jvm/jdk-11.0.6/bin/jconsole
jdk jdb /usr/lib/jvm/jdk-11.0.6/bin/jdb
jdk jdeps /usr/lib/jvm/jdk-11.0.6/bin/jdeps
jdk jinfo /usr/lib/jvm/jdk-11.0.6/bin/jinfo
jdk jmap /usr/lib/jvm/jdk-11.0.6/bin/jmap
jdk jps /usr/lib/jvm/jdk-11.0.6/bin/jps
jdk jrunscript /usr/lib/jvm/jdk-11.0.6/bin/jrunscript
jdk jstack /usr/lib/jvm/jdk-11.0.6/bin/jstack
jdk jstat /usr/lib/jvm/jdk-11.0.6/bin/jstat
jdk jstatd /usr/lib/jvm/jdk-11.0.6/bin/jstatd
jdk rmic /usr/lib/jvm/jdk-11.0.6/bin/rmic
jdk serialver /usr/lib/jvm/jdk-11.0.6/bin/serialver
```

Put it in the right position, under

```bash
/usr/lib/jvm/.jdk-11.0.6.jinfo
```

Right now, `update-alternatives` just detect it

```bash
stylee@matrix17:~$ sudo update-java-alternatives -l
java-1.11.0-openjdk-amd64      1111       /usr/lib/jvm/java-1.11.0-openjdk-amd64
jdk-11.0.6                     1106       /usr/lib/jvm/jdk-11.0.6
oracle-java6-jdk-amd64         316        /usr/lib/jvm/oracle-java6-jdk-amd64
oracle-java8-jdk-amd64         318        /usr/lib/jvm/oracle-java8-jdk-amd64
stylee@matrix17:~$ 
```

But it's not finished, yet: we have to tell `update-alternatives` which binary
to bind.

## Configure JDK 11 binaries to alternatives system

Launch the following commands

```bash
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk-11.0.6/bin/java 1106
sudo update-alternatives --install /usr/bin/jexec jexec /usr/lib/jvm/jdk-11.0.6/lib/jexec 1106
sudo update-alternatives --install /usr/bin/keytool keytool /usr/lib/jvm/jdk-11.0.6/bin/keytool 1106
sudo update-alternatives --install /usr/bin/pack200 pack200 /usr/lib/jvm/jdk-11.0.6/bin/pack200 1106
sudo update-alternatives --install /usr/bin/rmid rmid /usr/lib/jvm/jdk-11.0.6/bin/rmid 1106
sudo update-alternatives --install /usr/bin/rmiregistry rmiregistry /usr/lib/jvm/jdk-11.0.6/bin/rmiregistry 1106
sudo update-alternatives --install /usr/bin/unpack200 unpack200 /usr/lib/jvm/jdk-11.0.6/bin/unpack200 1106
sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/jdk-11.0.6/bin/jar 1106
sudo update-alternatives --install /usr/bin/jarsigner jarsigner /usr/lib/jvm/jdk-11.0.6/bin/jarsigner 1106
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk-11.0.6/bin/javac 1106
sudo update-alternatives --install /usr/bin/javadoc javadoc /usr/lib/jvm/jdk-11.0.6/bin/javadoc 1106
sudo update-alternatives --install /usr/bin/javap javap /usr/lib/jvm/jdk-11.0.6/bin/javap 1106
sudo update-alternatives --install /usr/bin/jcmd jcmd /usr/lib/jvm/jdk-11.0.6/bin/jcmd 1106
sudo update-alternatives --install /usr/bin/jconsole jconsole /usr/lib/jvm/jdk-11.0.6/bin/jconsole 1106
sudo update-alternatives --install /usr/bin/jdb jdb /usr/lib/jvm/jdk-11.0.6/bin/jdb 1106
sudo update-alternatives --install /usr/bin/jdeps jdeps /usr/lib/jvm/jdk-11.0.6/bin/jdeps 1106
sudo update-alternatives --install /usr/bin/jinfo jinfo /usr/lib/jvm/jdk-11.0.6/bin/jinfo 1106
sudo update-alternatives --install /usr/bin/jmap jmap /usr/lib/jvm/jdk-11.0.6/bin/jmap 1106
sudo update-alternatives --install /usr/bin/jps jps /usr/lib/jvm/jdk-11.0.6/bin/jps 1106
sudo update-alternatives --install /usr/bin/jrunscript jrunscript /usr/lib/jvm/jdk-11.0.6/bin/jrunscript 1106
sudo update-alternatives --install /usr/bin/jstack jstack /usr/lib/jvm/jdk-11.0.6/bin/jstack 1106
sudo update-alternatives --install /usr/bin/jstat jstat /usr/lib/jvm/jdk-11.0.6/bin/jstat 1106
sudo update-alternatives --install /usr/bin/jstatd jstatd /usr/lib/jvm/jdk-11.0.6/bin/jstatd 1106
sudo update-alternatives --install /usr/bin/rmic rmic /usr/lib/jvm/jdk-11.0.6/bin/rmic 1106
sudo update-alternatives --install /usr/bin/serialver serialver /usr/lib/jvm/jdk-11.0.6/bin/serialver 1106
```
and that's all, it's all configured correctly.

## Use your JDK 11.0.6

To use it, run the following

```bash
stylee@matrix17:~$ sudo update-java-alternatives -s jdk-11.0.6 
update-alternatives: errore: nessuna alternativa per iceweasel-javaplugin.so
update-alternatives: errore: nessuna alternativa per jaotc
update-alternatives: errore: nessuna alternativa per jdeprscan
update-alternatives: errore: nessuna alternativa per jhsdb
update-alternatives: errore: nessuna alternativa per jimage
update-alternatives: errore: nessuna alternativa per jlink
update-alternatives: errore: nessuna alternativa per jmod
update-alternatives: errore: nessuna alternativa per jshell
stylee@matrix17:~$
```

To prove your java executable is the right one

```bash
stylee@matrix17:~$ java -version
java version "11.0.6" 2020-01-14 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.6+8-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.6+8-LTS, mixed mode)
stylee@matrix17:~$ 
```