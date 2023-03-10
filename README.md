# Ansible-Event-Driven-Monitor


https://www.techbeatly.com/wp-admin/post.php?post=13560&action=edit

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

**use case**
- monitor the rpm installed 
- once the blocked rpm is detected, remove it


**Solution**

Use kafka event, Event-Driven Ansible and Ansible Rulebooks
- use apach kafka to monior the log file /var/log/dnf.log and generate the event for each new line from this log file
- use Event-Driven Ansible and Ansible Rulebooks to detect the blocked rpm. Once find it, remove it.

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

# grep -v "^#" config/connect-standalone.properties

bootstrap.servers=localhost:9092

key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=true
value.converter.schemas.enable=true

offset.storage.file.filename=/tmp/connect.offsets
offset.flush.interval.ms=10000

plugin.path=libs/connect-file-3.3.1.jar
```


```
# grep -v "^#" ./config/connect-file-source.properties 

name=local-file-source
connector.class=FileStreamSource
tasks.max=1
file=/var/log/dnf.log
topic=connect-dnf
```

```
# grep -v "^#" config/connect-file-sink.properties 

name=local-file-sink
connector.class=FileStreamSink
tasks.max=1
file=dnf.sink.txt
topics=connect-dnf
```

Next, we'll start two connectors running in standalone mode, which means they run in a single, local, dedicated process. We provide three configuration files as parameters. The first is always the configuration for the Kafka Connect process, containing common configuration such as the Kafka brokers to connect to and the serialization format for data. The remaining configuration files each specify a connector to create. These files include a unique connector name, the connector class to instantiate, and any other configuration required by the connector.

Open another terminal session and run:
```
bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```

Note that the data is being stored in the Kafka topic connect-dnf, so we can also run a console consumer to see the data in the topic (or use custom consumer code to process it):
```
# bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-dnf --from-beginning
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T04:51:21-0500 INFO --- logging initialized ---"}
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T04:51:21-0500 DDEBUG timer: config: 6 ms"}
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T04:51:22-0500 DEBUG Loaded plugins: builddep, changelog, config-manager, copr, debug, debuginfo-install, download, generate_completion_cache, groups-manager, needs-restarting, playground, product-id, repoclosure, repodiff, repograph, repomanage, reposync, subscription-manager, uploadprofile"}
...
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T18:21:37+0800 DEBUG Installed: checkpolicy-2.9-1.el8.x86_64"}
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T18:21:37+0800 DEBUG Installed: grub2-tools-efi-1:2.02-142.el8_7.1.x86_64"}
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T18:21:37+0800 DEBUG Installed: kernel-4.18.0-425.10.1.el8_7.x86_64"}
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T18:21:37+0800 DEBUG Installed: kernel-core-4.18.0-425.10.1.el8_7.x86_64"}
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T18:21:37+0800 DEBUG Installed: kernel-modules-4.18.0-425.10.1.el8_7.x86_64"}
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T18:21:37+0800 DEBUG Installed: policycoreutils-python-utils-2.9-20.el8.noarch"}
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T18:21:37+0800 DEBUG Installed: python3-audit-3.0.7-4.el8.x86_64"}
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T18:21:37+0800 DEBUG Installed: python3-libsemanage-2.9-9.el8_6.x86_64"}
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T18:21:37+0800 DEBUG Installed: python3-policycoreutils-2.9-20.el8.noarch"}
{"schema":{"type":"string","optional":false},"payload":"2023-01-14T18:21:37+0800 DEBUG Installed: python3-setools-4.3.0-3.el8.x86_64"}
...
```

The connectors continue to process data, so we can add data to the file and see it move through the pipeline:

```
$ echo Another line>> /var/log/dnf.log
..
```

You should see the line appear in the console consumer output.

https://www.redhat.com/sysadmin/introduction-tmux-linux

# Result

**Start ansible-rulebook to monitor events**

```
[jinzha@localhost kafka]$ ansible-rulebook --rulebook kafka-dnf.yml -i inventory.yml --verbose
INFO:ansible_rulebook.app:Starting sources
INFO:ansible_rulebook.app:Starting rules
INFO:ansible_rulebook.engine:run_ruleset
INFO:ansible_rulebook.engine:ruleset define: {"name": "Read messages from a kafka topic and act on them", "hosts": ["kafka.example.com"], "sources": [{"EventSource": {"name": "ansible.eda.kafka", "source_name": "ansible.eda.kafka", "source_args": {"host": "kafka.example.com", "port": 9092, "topic": "connect-dnf", "group_id": null}, "source_filters": []}}], "rules": [{"Rule": {"name": "receive event", "condition": {"AllCondition": [{"EqualsExpression": {"lhs": {"Event": "schema.type"}, "rhs": {"String": "string"}}}]}, "action": {"Action": {"action": "run_playbook", "action_args": {"name": "dnf-event.yaml"}}}, "enabled": true}}]}
INFO:ansible_rulebook.engine:load source
INFO:ansible_rulebook.engine:load source filters
INFO:ansible_rulebook.engine:Calling main in ansible.eda.kafka
INFO:aiokafka.consumer.subscription_state:Updating subscribed topics to: frozenset({'connect-dnf'})
INFO:ansible_rulebook.engine:Waiting for event from Read messages from a kafka topic and act on them
INFO:aiokafka.consumer.group_coordinator:Metadata for topic has changed from {} to {'connect-dnf': 1}. 
...

```

**Install telnet**

![Install telnet](images/ansible-eda-01.png)
`telnet` package is installed.

**Trigger the playbook**

The events are listened. When telnet package was found, task `Remove the RPM listed in black list` is trigger.
![Install telnet](images/ansible-eda-02.png)

**telnet is removed**

![Install telnet](images/ansible-eda-03.png)

