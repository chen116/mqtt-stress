# Stress Test Scripts for MQTT using Python and Ansible

*Scripts* that try to stress test a messaging broker based on MQTT for assessment purposes.

### Currently implemented

Python script that creates a bunch of threads: each thread is then instructed to (1) wait a random delay (milliseconds between limits) (2) publish one message (random string with chars and numbers) to a MQTT broker, using a topic built on the same random string, and (3) go idle again. On a main loop, a number of random messages are created and passed to the idle threads with a queue. A subscription is made at the same broker and received messages are counted (no check on the content).

The effect is: a numbered sequence of random messages, being published, with a certain degree of superimposition. `CTRL^C` interrupts (but waits for all pending threads to exit). At the end of the sequence statistics are given about how many messages have been generated and how many received (purposely the same number).

Configuration is contained in a INI-styled file.

```
## install prerequisites ##
# in Ubuntu/Debian
user@host:~$ sudo aptitude install python-pip
user@host:~$ sudo pip install paho-mqtt
# or Fedora/Centos
user@host:~$ sudo yum install python-pip
user@host:~$ sudo pip install paho-mqtt

## clone this script and execute against test.mosquitto.org (do not abuse!) ##
user@host:~$ git clone https://github.com/bluewindthings/mqtt-stress
user@host:~$ cd mqtt-stress
user@host:~/mqtt-stress$ python mqtt_stress_test

## configure for your server / options ##
user@host:~/mqtt.stress$ vi mqtt_stress_test.ini
```

Typical output would be:

```
Config read from: mqtt_stress_test.ini

==================================
url       : test.mosquitto.org:1883
clientid  : stress
msg_num   : 900
sleep_min : 0.5
sleep_max : 1.5
threads   : 80
topic     : stress/fromlocalhost
msg       : test_ini
==================================

Trying url: mqtt://test.mosquitto.org:1883
Disconnect just in case
Now connecting...
Connected with rc: 0
Subscribed: 1 (0,)
Messages Generated: 900  Published: 660  Received: 652  so far, waiting publishing...
...waiting 5 more seconds for subcriptions to expire...
Disconnecting...
Messages Generated: 900  Published: 900  Received: 900
Elapsed Time: 17.1680960655
```

### Ansible

Under `ansible/` folder there's a set of configuration files for [Ansible](http://www.ansible.com) (playbooks) that makes it possible to deploy and run the test script and its configuration to a series of local/remote hosts. Configuration can be defined once for all, at host level, at run level.

Please note that before running the [Ansible](http://www.ansible.com) scripts you will need to configure for your needs, in terms of hosts addresses and test script parameters. Look for file names ending with `.sample` under `ansible/` folder and below, replicate them without `.sample` and modify according to your needs.

The [Ansible](http://www.ansible.com) scripts will try to copy to target hosts the stress test script mentioned above, and its configuration file built after specified variables. Will then run the script and collect result log files copied inside a new folder `ansible/fetched`. The stress test script and configuration file are deleted from target hosts at the end.

The main configuration points are:

* `ansible/production.sample`, `ansible/stage.sample`, `ansible/simplepub.sample`: define targets hostnames for different scenarios: production, staging (test), very simple one shot publication
* `ansible/roles/mqtt/defaults/main.yml`: configuration variables for the test script, once for all (default if not specified anywhere)
* `ansible/host_vars/*.sample`: (one file per host named exactly after its hostname) SSH access, sudo password, test script configuration variables relevant to the single target host

Run with:

```
yourname@host:~/things-tests$ ansible-playbook -i ansible/stage ansible/site.yml
yourname@host:~/things-tests$ ansible-playbook -i ansible/stage ansible/simplepub.yml
yourname@host:~/things-tests$ ansible-playbook -i ansible/production ansible/site.yml
yourname@host:~/things-tests$ ansible-playbook -i ansible/production ansible/cleanup.yml
```

### Contact

* Author: [BlueWind] Stefano Costa (http://www.bluewind.it)
* Email: stefano DOT costa AT bluewind DOT it

