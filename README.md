# Oceannik's Deployment Strategies

## Available Deployment Strategies

- `blue-green` performs a complete blue-green deployment.
- `debug` is used for testing. Does not mutate the host systems.

## Required collections

To install all the required collections, issue the following command:

```
ansible-galaxy install -r requirements.yml
```

If the collections fail to install, make sure that you're running an up-to-date version of ansible.

## Tested versions

The playbooks were tested on `ansible-core` version `2.11.4`.

## Roles

### preconfigure_hosts

Preconfigures all given hosts so the deployment can be executed. 
This includes creating a new `oceannik` group, and the `oceannik` user with a home directory.
The home directory on the master host contains a `projects/` directory that stores status information about deployments.
The `oceannik` user is used for performing actions during the deployment process.
Additionally, the role installs tools such as `Python`, `pip`, `Docker`, `nginx`, and more.
On the master host, it sets up the firewall using `ufw`.

### gather_system_details

Fetches a project status file from the master host.
The file contains information about the active environment, as well as a list of created and active services.
If no such file exists, assumes a fresh deployment on a clean node.

### deploy_service

Performs all steps to deploy a single service. This includes settings the correct service name, executing roles for creating the container, configuring TLS certificates and generating new nginx configuration. If all steps are completed successfully, restarts the nginx service on the master host to apply the new configuration and to switch the application to the new environment.

### create_service_container_network

Creates the container network for services to be deployed in the new environment.

### create_service_container

Configures and starts the container as defined in the `oceannik.yml` service configuration file. Inspects the created container so it can fetch the randomly assigned host ports to the published container ports. Saves information about the created services and their configuration that will then later be used to create a project status file.

### configure_service_tls 

Checks available SSL/TLS certificates for a domain name set by the user. If certificates do not exist yet, and the user has agreed to generate them in `oceannik.yml`, proceeds with generating by configuring *certbot*. *certbot* then obtains certificates from Let's Encrypt.

### configure_service_access

Fetches the port of the service's container, generates a new nginx config file for the service and checks the syntax of the created config file. This role allows the container to be exposed to the Internet under a domain set by the user. Depending on the user configuration, the container will be hosted using *https*, or *http* only.

### check_service_status

Checks whether the published service container is active by sending an HTTP GET request checking whether the returned status code was 200.
This check is performed 6 times with a delay of 5 seconds, which means that the application has 30 seconds to get configured.
