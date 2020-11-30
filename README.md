# ChirpStack Docker With Mosquitto Auth
This repository contains a skeleton to setup the [ChirpStack](https://www.chirpstack.io)
open-source LoRaWAN Network Server stack using [Docker Compose](https://docs.docker.com/compose/).

## Modifications in Mosquitto docker to have minimal atentification

Added a preconfigured strucure for Mosquitto auth. Password static method. "Static password and ACL file" [Mqtt Auth](https://www.chirpstack.io/project/guides/mqtt-authentication/).

   Add to the docker-compose.yml the volume structure for mosquitto " - ./configuration/mosquitto:/mosquitto "

   Data Origen, the folder where you have the docker-compose.yml file --> Configuration folder, to match with the other conf files for Chirpstack

      - Configuration
         Inside you will see:
            File passwd: user + password (you need to replace use&password) 
            File acls: acces control list by user (you need to replace users, and if you whant more restrictions, change it based in mosquitto acces rules)
            Folder config:
               File mosquitto.conf
                     "allow_anonymous false
                     password_file /mosquitto/passwd
                     acl_file /mosquitto/acls"
   Data destination

-	This Is the volume folder where will be synchronised the config data. by default is mosquitto, and inside will be synchronized the folders and files from Data origen.

-	Chirpstack conf.
	Is needed to setup each of the following configurations to allow applications to see each other and with mosquitto.
	Set the user and password.	 

		chirpstack-application-server.toml
		chirpstack-gateway-bridge.toml
		chirpstack-network-server.toml


* Password generation

	To be able to generate secure password you need to go to the mosquitto docker container and execute inside the mosquitto password commands.

	Enter to the container:
	
	    sudo docker exec -it <ID container> sh
	Once inside the container generate your password

	    mosquitto_passwd -c /mosquitto/passwd yourUSER
User and password crypted generated will be stored in "passwd" file and synchronized with the volume before created. 

Note:
- The users created in this step need to be included in the "acls" file.
- The password used in chirpstack configuration files ARE NOT the crypted password generated inside mosquitto, need to be the original password used.



**Note:** Please use this `docker-compose.yml` file as a starting point for testing
but keep in mind that for production usage it might need modifications. 

## Directory layout

* `docker-compose.yml`: the docker-compose file containing the services
* `docker-compose-env.yml`: alternate docker-compose file using environment variables, can be run with the docker-compose `-f` flag
* `configuration/chirpstack*`: directory containing the ChirpStack configuration files, see:
    * https://www.chirpstack.io/gateway-bridge/install/config/
    * https://www.chirpstack.io/network-server/install/config/
    * https://www.chirpstack.io/application-server/install/config/
    * https://www.chirpstack.io/geolocation-server/install/config/
* `configuration/postgresql/initdb/`: directory containing PostgreSQL initialization scripts

## Configuration

The ChirpStack stack components components are pre-configured to work with the provided
`docker-compose.yml` file and defaults to the EU868 LoRaWAN band. Please refer
to the `configuration/chirpstack-network-server/examples` directory for more configuration
examples.

# Data persistence

PostgreSQL and Redis data is persisted in Docker volumes, see the `docker-compose.yml`
`volumes` definition.

## Requirements

Before using this `docker-compose.yml` file, make sure you have [Docker](https://www.docker.com/community-edition)
installed.

## Usage

To start the ChirpStack open-source LoRaWAN Network Server stack, simply run:

```bash
$ docker-compose up
```

**Note:** during the startup of services, it is normal to see the following errors:

* ping database error, will retry in 2s: dial tcp 172.20.0.4:5432: connect: connection refused
* ping database error, will retry in 2s: pq: the database system is starting up


After all the components have been initialized and started, you should be able
to open http://localhost:8080/ in your browser.

### Add Network Server

When adding the Network Server in the ChirpStack Application Server web-interface
(see [Network Servers](https://www.chirpstack.io/application-server/use/network-servers/)),
you must enter `chirpstack-network-server:8000` as the Network Server `hostname:IP`.
