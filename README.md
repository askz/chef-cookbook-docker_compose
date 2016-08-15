# Docker Compose Cookbook

The Docker Compose Cookbook is a library cookbook that provides custom
resources for use in recipes.


## Requirements

- Working Docker installation. You might want to use the excellent
[docker Cookbook](https://supermarket.chef.io/cookbooks/docker) to provision
Docker.


## Usage

Place a dependency on the docker-compose cookbook in your cookbook's
metadata.rb 

```ruby
depends 'docker_compose', '~> 0.0'
```

Create a [Docker Compose file](https://docs.docker.com/compose/compose-file/)
for the application you want to provision. A simple Compose file that uses the
[official nginx Docker image](https://hub.docker.com/_/nginx/) looks like this:

```
version: '2'
services:
  web_server:
    image: nginx
    ports:
      - "80:80"
```

Then, in a recipe:

```ruby
include_recipe 'docker_compose::installation'

# Provision Compose file
cookbook_file '/etc/docker-compose_nginx.yml' do
  source 'docker-compose_nginx.yml'
  owner 'root'
  group 'root'
  mode 0640
  notifies :up, 'docker_compose_application[nginx]', :delayed
end

# Provision Compose application
docker_compose_application 'nginx' do
  action :up
  compose_files [ '/etc/docker-compose_nginx.yml' ]
end
```

## Attributes

- `node['docker_compose']['release']` - The release version of Docker Compose
 to install. Defaults to a sane, current default.

- `node['docker_compose']['command_path']` - The path under which the
 `docker-compose` command should be installed.
 Defaults to `/usr/local/bin/docker-compose`

## Resources Overview
 
### docker_compose_application

The `docker_compose_application` provisions a Docker application (that usually
consists of several services) using a Docker Compose file.

#### Example

```ruby
docker_compose_application 'nginx' do
  action :up
  compose_files [ '/etc/docker-compose_nginx.yml', '/etc/docker-compose_nginx.additional.yml' ]
end
```

#### Parameters

- `project_name` - A string to identify the Docker Compose application.
 Defaults to the resource name.

- `compose_files` - The list of Compose files that makes up the Docker Compose
 application. The specified file names are passed to the `docker-compose`
 command in the order in which they appear in the list.
 
#### Actions

- `:up` - Create and start containers.
  Equivalent to calling `docker-compose up` with the Compose files specified
  using the `compose_files` parameter.
 
- `:down` - Stop and remove containers, networks, images, and volumes.
  Equivalent to calling `docker-compose down` with the Compose files specified
  using the `compose_files` parameter.

## License & Authors

- Author:: Sebastian Boschert (<sebastian@2007.org>)
