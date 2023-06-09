# AEM Docker Stack Environment

The AEM Docker Stack Environment is a simple way to get a full 1-1-1 (Author - Publish - Dispatcher) Adobe AEM stack running in docker.
It's very similar to the AMS (Adobe Managed Services) environment, and the Dockerfiles were written to allow the installation of packages the first time you run the instances. 

Since AMS uses the RHEL distro, the dispatcher is built on top of Centos7 and the aem-base image (from which the author and publish inherit from) are based on the [azul/zulu-openjdk:17-latest](https://hub.docker.com/r/azul/zulu-openjdk) which at the time of writing is the latest OpenJDK LTS (release 17).

The project structure is as follows:

| Structure  |                                                  |
|------------|--------------------------------------------------|
| aem-base   | Base image that contains the aem-quickstart-$version.jar and the license.properties file and initial bootstap packages |
| author     | Builds the aem-base image and sets it to the author run-mode |
| publish    | Builds the aem-base image and sets it to the publish run-mode |
| dispatcher | Builds an apache server with the dispatcher module and an haproxy server to mimic SSL |


## Disclaimer
Adobe Experience Manager (AEM) is a proprietary software developed and owned by Adobe Systems. It is a commercial product and requires a license to use. As a proprietary solution, the source code of AEM is not publicly available, and only Adobe has the rights to modify and distribute the software. Organizations that wish to use AEM must enter into a licensing agreement with Adobe to obtain the necessary permissions and access to the software.

# Basic Setup

1 - Clone the repository to your local machine

```shell
$ git clone {project url}
```
2 - Copy the adobe proprietary (See disclaimer above) ```aem-quickstart-6.5.0.jar``` and ```license.properties``` files into the root of the ```aem-base``` folder. Make sure the files are named exactly like that.
If you wish to use different filenames you can set these in the arg variable ```AEM_FILE```

2.1 (Optional) Copy any additional AEM packages into the ```packages``` folder you want to install in your AEM instaces (i.e. across all run modes). For instance, I like to install the latest service packs, ACS Commons and some demo packages like the "wknd website" that later help me during development. So I copy the packages (zip) files into the ```aem-base\packages``` folder to ensure every run mode has these packages, and the "x-ray" package (zip) into the ```author\packages``` folder, because it doesn't make sense to put it in the publisher (at least for me).

3 - If you do not have a valid dispatcher configuration in your project and want one that matches the AMS cloud setup and that starts running immediately , run the script:

```shell
$ chmod +x ./dispatcher/scripts/init-dispatcher-config.sh
$ ./dispatcher/init-dispatcher-config.sh
```
If however you do have a dispatcher configuration available in your project, simply bind your project's local path to the container paths (as shown in the docker-compose.yaml below).

4 - Check all environment variables in ```.env.dispatcher.sh``` and change accordingly. The defaults are as follows:

```
DISP_ID=docker
AUTHOR_IP=192.168.10.10 (your ip)
AUTHOR_PORT=4502
AUTHOR_DEFAULT_HOSTNAME=author.docker.local
AUTHOR_DOCROOT=/mnt/var/www/author
PUBLISH_IP=192.168.10.10 (your ip)
PUBLISH_PORT=4503
PUBLISH_DEFAULT_HOSTNAME=publish.docker.local
PUBLISH_DOCROOT=/mnt/var/www/html
LIVECYCLE_IP=127.0.0.1
LIVECYCLE_PORT=8080
LIVECYCLE_DEFAULT_HOSTNAME=aemforms-exampleco-dev.adobecqms.net
LIVECYCLE_DOCROOT=/mnt/var/www/lc
PUBLISH_FORCE_SSL=0
AUTHOR_FORCE_SSL=0
CRX_FILTER=deny
DISPATCHER_FLUSH_FROM_ANYWHERE=allow
ENV_TYPE=dev
RUNMODE=sites
```
5 - Launch the full stack with:

```shell
$ docker-compose up -d
```

A ```docker-compose.yaml``` is provided, but you can of course modify it to your needs. 
It may take a while for the entire stack to finish the boot the first time, especially if you have many packages in the ```.../packages``` folder.
Once ready, test your configuration with:
- author - {yourip}:4502 (e.g. http://192.168.10.10:4502)
- publish - {yourip}:4503 (e.g. http://192.168.10.10:4503)
- dispatcher - http://we-retail.docker.local/content/we-retail/language-masters/en.html (***)

# Configure your localhost file (***)
In order for the dispatcher to work correctly, it is advisable to configure your ```hosts``` file accordingly. 

| OS      |                                       |
|---------|---------------------------------------|
| Windows | c:\Windows\System32\Drivers\etc\hosts |
| Linux   | /etc/hosts |

```
127.0.0.1 author.docker.local
127.0.0.1 publish.docker.local
127.0.0.1 we-retail.docker.local
127.0.0.1 host.docker.internal
```
# Docker Compose volume bindings

As mentioned above, you can adjust the docker-compose volume bindings. You can either copy your dispatcher configuration inside the default paths (AMS), or bind new ones to the container default location:

- Default Configuration
```
    volumes:
    # Dispatcher configuration
      - ./dispatcher/ams/2.6/etc/httpd/conf:/etc/httpd/conf:ro
      - ./dispatcher/ams/2.6/etc/httpd/conf.d:/etc/httpd/conf.d:ro
      - ./dispatcher/ams/2.6/etc/httpd/conf.dispatcher.d:/etc/httpd/conf.dispatcher.d:ro
      - ./dispatcher/ams/2.6/etc/httpd/conf.modules.d:/etc/httpd/conf.modules.d:ro

    # Caching      
      - ./dispatcher/mnt/author_docroot:/mnt/var/www/author:rw
      - ./dispatcher/mnt/publish_docroot:/mnt/var/www/html:rw
      
    # Logs   
      - ./dispatcher/mnt/log:/var/log/httpd:rw
```
- Custom Project Config

```
    volumes:
    # Dispatcher configuration
      - /{my_project_location}/dispatcher/src/conf:/etc/httpd/conf:ro
      - /{my_project_location}/dispatcher/src/conf.d:/etc/httpd/conf.d:ro
      - /{my_project_location}/dispacther/src/conf.dispatcher.d:/etc/httpd/conf.dispatcher.d:ro
      - /{my_project_location}/dispatcher/src/conf.modules.d:/etc/httpd/conf.modules.d:ro

    # Caching      
      - /{my_project_location}/author_docroot:/mnt/var/www/author:rw
      - /{my_project_location}/publish_docroot:/mnt/var/www/html:rw
      
    # Logs   
      - /{my_project_location}/my_log:/var/log/httpd:rw

```

Remember that if you change Apache/Dispatcher configuration, you must restart the apache. To access the dispatcher shell do:

```shell
docker exec -it dispatcher /bin/bash
```
or

```shell
docker restart -t0 dispatcher
```
The last command kills the container straight away without waiting for it to finish gracefully.













