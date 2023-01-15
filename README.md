# Ansible-Event-Driven-Monitor

```
echo "# Ansible-Event-Driven-Monitor" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/alpha-wolf-jin/Ansible-Event-Driven-Monitor.git
git config --global credential.helper 'cache --timeout 72000'
git push -u origin main

git add . ; git commit -a -m "update README" ; git push -u origin main
```

# Install ansible-rulebook

**Install the latest RHEL 9**

**Install ansible-rulebook**

Reference: https://github.com/ansible/ansible-rulebook/blob/main/docs/installation.rst

```
dnf --assumeyes install gcc java-17-openjdk maven python3-devel python3-pip
export JDK_HOME=/usr/lib/jvm/java-17-openjdk
export JAVA_HOME=$JDK_HOME
pip3 install -U Jinja2
pip3 install ansible ansible-rulebook ansible-runner wheel
pip3  install aiokafka
```

**Install collection ansible.eda**

```
ansible-galaxy collection install community.general ansible.eda
```

# Install APACHE KAFKA

Reference https://kafka.apache.org/quickstart

Download kafka from below link:
https://www.apache.org/dyn/closer.cgi?path=/kafka/3.3.1/kafka_2.13-3.3.1.tgz

## extract install tgz file
```
$ tar -xzf kafka_2.13-3.3.1.tgz
$ cd kafka_2.13-3.3.1
```

## Start Kafka with ZooKeeper
### Start the ZooKeeper service
Run the following commands in order to start all services in the correct order:
```
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

### Start the Kafka broker service
Open another terminal session and run:
```
$ bin/kafka-server-start.sh config/server.properties
```

# IMPORT/EXPORT YOUR DATA AS STREAMS OF EVENTS WITH KAFKA CONNECT

First, make sure to add connect-file-3.3.1.jar to the plugin.path property in the Connect worker's configuration.

Edit the config/connect-standalone.properties file, add or change the plugin.path configuration property match the following, and save the file:
```
$ echo "plugin.path=libs/connect-file-3.3.1.jar"
```

Then, start by creating some seed data to test with:
```
$ echo -e "foo\nbar" > test.txt
```

Note that the data is being stored in the Kafka topic connect-test, so we can also run a console consumer to see the data in the topic (or use custom consumer code to process it):
```
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
...
```

The connectors continue to process data, so we can add data to the file and see it move through the pipeline:

```
$ echo Another line>> test.txt
..
```

You should see the line appear in the console consumer output and in the sink file.


