---
uid: InstallPIAdapterForModbusTCPUsingDocker
---

# Installation using Docker

Docker is a set of tools that you can use on Linux to manage application deployments.

**Note:** If you want to use Docker, you must be familiar with the underlying technology and have determined that it is appropriate for your planned use of the Modbus TCP adapter. Docker is not required to use  the Modbus TCP adapter.

This topic provides examples of how to create a Docker container with the Modbus TCP adapter.

## Create a startup script for the adapter

1. Use a text editor to create a script similar to one of the following examples:

	**Note:** The script varies slightly by processor.

	**ARM32**

	```bash
	#!/bin/sh
	if [ -z $portnum ] ; then
		exec /PI-Adapter-for-Modbus_1.3.0.131-arm_/OSIsoft.Data.System.Host
	else
		exec /PI-Adapter-for-Modbus_1.3.0.131-arm_/OSIsoft.Data.System.Host --port:$portnum
	fi
	```

	**ARM64**

	```bash
	#!/bin/sh
	if [ -z $portnum ] ; then
		exec /PI-Adapter-for-Modbus_1.3.0.131-arm64_./OSIsoft.Data.System.Host
	else
		exec /PI-Adapter-for-Modbus_1.3.0.131-arm64_/OSIsoft.Data.System.Host --port:$portnum
	fi
	```

	**AMD64**
			
	```bash
	#!/bin/sh
	if [ -z $portnum ] ; then
		exec /PI-Adapter-for-Modbus_1.3.0.131-x64_/OSIsoft.Data.System.Host
	else
		exec /PI-Adapter-for-Modbus_1.3.0.131-x64_/OSIsoft.Data.System.Host --port:$portnum
	fi
	```
	
2. Name the script `modbusdockerstart.sh` and save it to the directory where you plan to create the container.

## Create a Docker container containing the Modbus TCP adapter

1. Create the following `Dockerfile` in the directory where you want to create and run the container.

	**Note:** `Dockerfile` is the required name of the file. Use the variation according to your operating system:

	**ARM32**

	```bash
	FROM ubuntu
	WORKDIR /
	RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y ca-certificates libicu60 libssl1.1 curl
	COPY modbusdockerstart.sh /
	RUN chmod +x /modbusdockerstart.sh
	ADD ./PI-Adapter-for-Modbus_1.3.0.131-arm_.tar.gz .
	ENTRYPOINT ["/modbusdockerstart.sh"]
	```
	**ARM64**

	```bash
	FROM ubuntu
	WORKDIR /
	RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y ca-certificates libicu66 libssl1.1 curl
	COPY modbusdockerstart.sh /
	RUN chmod +x /modbusdockerstart.sh
	ADD ./PI-Adapter-for-Modbus_1.3.0.131-arm64_.tar.gz .
	ENTRYPOINT ["/modbusdockerstart.sh"]
	```

	**AMD64 (x64)**

	```bash
	FROM ubuntu
	WORKDIR /
	RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y ca-certificates libicu66 libssl1.1 curl
	COPY modbusdockerstart.sh /
	RUN chmod +x /modbusdockerstart.sh
	ADD ./PI-Adapter-for-Modbus_1.3.0.131-x64_.tar.gz .
	ENTRYPOINT ["/modbusdockerstart.sh"]
	```

2. Copy the `Modbus_linux-\<platform>.tar.gz` file to the same directory as the `Dockerfile`.

3. Copy the `modbusdockerstart.sh` script to the same directory as the `Dockerfile`.

4. Run the following command line in the same directory (`sudo` may be necessary):

	```bash
	docker build -t modbusadapter .
	```

## Run the Modbus TCP adapter Docker container

### REST access from the local host to the Docker container

Complete the following steps to run the container:

1. Use the docker container image `modbusadapter` created previously.
2. Type the following in the command line (`sudo` may be necessary):

	```bash
	docker run -d --network host modbusadapter
	```

Port `5590` is accessible from the host and you can make REST calls to Modbus TCP adapter from applications on the local host computer. In this example, all data stored by the Modbus TCP adapter is stored in the container itself. When you delete the container, the stored data is also deleted.

### Provide persistent storage for the Docker container

Complete the following steps to run the container:

1. Use the docker container image `modbusadapter` created previously.
2. Type the following in the command line (`sudo` may be necessary):

	```bash
	docker run -d --network host -v /modbus:/usr/share/OSIsoft/ modbusadapter
	```

Port `5590` is accessible from the host and you can make REST calls to Modbus TCP adapter from applications on the local host computer. In this example, all data that is written to the container is instead written to the host directory and the host directory is a directory on the local machine, `/modbus`. You can specify any directory.

### Port number change

To use a different port other than `5590`, you can specify a `portnum` variable on the `docker run` command line. For example, to start the adapter using port `6000` instead of `5590`, use the following command:

```bash
docker run -d -e portnum=6000 --network host modbusadapter
```

This command accesses the REST API with port `6000` instead of port `5590`. The following `curl` command returns the configuration for the container.

```bash
curl http://localhost:6000/api/v1/configuration
```

### Remove REST access to the Docker container

If you remove the `--network host` option from the docker run command, REST access is not possible from outside the container. This may be of value where you want to host an application in the same container as OPC UA adapter but do not want to have external REST access enabled
