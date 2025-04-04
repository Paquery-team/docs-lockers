# Ansible-playbooks

[Ansible](https://docs.ansible.com/ansible/latest) playbooks for Raspberry and Odroid H2 operations.

> NOTE:
> folders for Odroid H2 have "Odroid" at the end of the folder name.

## Installation

To install Ansible on host, first run:

`sudo sh -c 'echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main" >> /etc/apt/sources.list'`

then:

```shell script
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
$ sudo apt update
$ sudo apt install -y ansible
```

You should make sure the ansible version you just installed is at least 2.9,
otherwise the playbooks may not function correctly:

```shell script
$ ansible --version
```

## Configuration

To download all the playbooks:

```shell script
$ ansible-pull -d /path/to/ansible/dir -U https://git_user:git_pass@gitlab.com/clicknbox/firmware/ansible-playbooks.git \
 /microservices/local/setup.yml -e @vars.json
```
When there are remote hosts

```shell script
$ ansible-ansible -d /path/to/ansible/dir -U https://git_user:git_pass@gitlab.com/clicknbox/firmware/ansible-playbooks.git \
 /microservices/hosts/setup.yml -e @vars.json
```

where `@vars.json` has the following properties:

```json
{
  "git_user": "firmware",
  "git_pass": "<deploy token to use>",
  "base_path": "<working dir>",
}
```
Add the variable ` "hosts_group": "<target groups to update>" ` when use the remote hosts

The `base_path` usually is `/home/pi/Documents`.

## Run a playbook

To execute a playbook:

```shell script
$ ansible-playbook path/to/playbook.yml -e @vars.json
```

where `vars.json` is in the same directory and contains all required variables. Note that is preferred to use
a file instead of passing secrets on the terminal.

> `sudo` is not required to run a playbook since it will attempt to become root for the required steps only.
> If running locally, you can pass `--ask-become-pass` to allow sudo actions.

## Playbooks

### Initialization

This scripts run the first setup: wifi connection, retrieve secrets, configure rc.local and software dependencies.

TODO

### Microservices

These playbooks install, update and migrate RPi moleculer services. The following variables are used:

**Node settings**

- `mqtt_user`, `mqtt_pass`, `mqtt_host`: authentication settings for MQTT broker
- `env`: environment to join (prod, stage)

**Services settings**
- `log_level`: log level output (one of: _fatal, error, warn, info_ (default), _debug_ or _trace_)
- `rpi_port`: local server port (exposed to the tablet, defaults to 3000)
- `job_minute`,`job_hour`,`job_day`: Cron tabs for events sync
- `rpi_serial`: serial port to use
- `db`: path to sqlite database file (defaults to: `{{env}}.sqlite`)

**Integration settings**
- `api_url`: Main API host
- `api_token`,`api_user`,`api_pass`: Secrets for API OAuth
- `particle_mail`,`particle_pass`,`particle_token_duration`: Particle authentication settings

**Installation settings**
- `uuid`: location UUID to associate
- `version`: Tag version to install

#### Migration

[migration](microservices/migration.yml) playbook takes an installation where both [tower-rpi](../tower-rpi)
and [ms-rpi](../ms-rpi) exist. It will copy the environment variables and pull changes for `ms-rpi`.

The following variables are needed:

```json
{
    "env": "prod",
    "mqtt_user": "<mqtt_user>",
    "mqtt_pass": "<mqtt_pass>",
    "mqtt_host": "<mqtt_host>",
    "log_level": "info",
    "base_path": "/home/pi/Documents",
    "version": "<tag_to_install>"
  }
```

Note that all other variables will be taken from [tower-rpi configuration](../tower-rpi/.env.example).

#### First installation

[install](microservices/<Hosts or Local>/install.yml) playbook will install node dependencies
and pull changes for [ms-rpi](../ms-rpi) project, setting all environment
variables and creating a systemd service to manage it.
The following variables are needed:

```json
{
    "env": "prod",
    "mqtt_user": "<broker_user>",
    "mqtt_pass": "<broker_pass>",
    "mqtt_host": "<broker_host>",
    "job_minute": "*/5 * * * *",
    "job_hour": "0 * * * *",
    "job_day": "0 4 * * *",
    "db": "some.db",
    "log_level": "info",
    "rpi_serial": "/dev/ttyUSB",
    "rpi_port": 3000,
    "api_url": "http://api.prod", 
    "client_id": "<client_id>",
    "client_secret ": "<client_secret>",
    "particle_mail": "$PARTICLE_MAIL",
    "particle_pass": "$PARTICLE_PASS",
    "particle_token_duration": 3000,
    "uuid": "<location_uuid>",
    "version": "<tag_to_install>",
    "git_pass": "$GIT_DEPLOY_TOKEN",
    "git_user": "$GIT_DEPLOY_USER",
    "base_path": "/home/pi/Documents",
    "hosts_group": "<target groups to update>"
}
```

#### Updates

[update](microservices/<Hosts or Local>/update.yml) playbook will pull changes for [ms-rpi](../ms-rpi) and
restart the service. The following variables are needed:

Add the variable ` "hosts_group": "<target groups to update>" ` when use the remote hosts

```json
{
    "version": "<tag_to_install>",
    "API_BASE_URL": "<new version of api>"
}
```

### Dependencies

#### Bish-bosh

[bish-bosh](dependencies/bishbosh.yml) will install [bish-bosh MQTT client](https://github.com/raphaelcohn/bish-bosh)
and setup a service to manage the [shell client](../mqtt-shell-client). The required variables are:

```json
{
  "mqtt_user": "<broker_user>",
  "mqtt_pass": "<broker_pass>",
  "mqtt_host": "<broker_host>",
  "ansible_path": "/home/pi/Documents/ansible-playbooks",
  "uuid": "<locker_uuid>"
}
```

#### System upgrade

[system-upgrade](dependencies/system_upgrade.yml) will update and upgrade apt modules, will also install
node if a `node_version` variable is passed.

## Microservices (Odroid)

Ansible playbooks for installing microservices from "MS-rpi" repo on Odroid H2.

**Requirements**

To install ansible on host do:

```shell script
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

You should check MS-rpi repo for the requirements to run microservices as this is only to run the Ansible playbooks and not Moleculer microservices.

**Configuration**

And run on host:

```shell script
$ ansible-pull -o -d %variable1% -U https://%variable2%:%variable3%@gitlab.com/clicknbox/firmware/ansible-playbooks.git --full %variable#%/microservices/<Hosts or Local>/setup.yml -e "GITUSER=%variable4% GITPASS=%variable5% USER=%variable6% PASSWORD=%variable7% HOST=%variable8% CLIENT_ID=%variable9% NAMESPACE=%variable10% NODEID=%variable11% LOG_LEVEL=%variable12% DB=%variable13% SUDOPASS=%variable14% "
```

Note that you have to configure all the variables marked like: `%variable#%`. And this script is for an Odroid called "clicknbox", you should change this if your Odroid is named differently.

The following is a list of what each variable is:
- variable1 = path where ansible-palybooks repo will be cloned
- variable2 = Gitlab user for ansible-playbooks repo
- variable3 = Gitlab password for ansible-playbooks repo
- variable4 = Gitlab user for ms-rpi repo
- variable5 = Gitlab password for ms-rpi repo
- variable6 = USER for .env file
- variable7 = PASSWORD for .env file
- variable8 = HOST for .env file
- variable9 = CLIENT_ID for .env file
- variable10 = NAMESPACE for .env file
- variable11 = NODEID for .env file
- variable12 = LOG_LEVEL for .env file
- variable13 = DB for .env file
- variable14 = SUDO password for specific Odroid
