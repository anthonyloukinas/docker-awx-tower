# docker-awx-tower

This repository is a basic docker-compose configuration that works for both AWX and Ansible Tower using the new quay Tower images. I had to split the two configurations into their own directories due to issues with docker-compose overwriting the variations. I'm in the process of working on a better more dynamic solution using Ansible to configure these environments, but I couldn't find a *working* example of Ansible Tower/AWX just using compose files in Docker.

## Usage

For the AWX/Tower web container definitions in each respective compose file, you may want to modify the exposed HTTP port from the default "80", especially if you're planning on running multiple instances.

To run multiple instances, I would for now copy the tower/awx directories into new directories, and modify the compose yaml accordingly ensuring you use different HTTP ports.

### AWX

```bash
# Create AWX Database Volume
docker volume create awx_web

# Change to AWX directory
cd awx

# Initialize the AWX application in the background denoted by -d
docker-compose -f awx-compose.yml up -d
```

### Tower

```bash
# Create Tower Database Volume
docker volume create tower_web

# Change to Tower directory
cd tower

# Initialize the Tower application in the background denoted by -d
docker-compose -f tower-compose.yml up -d
```

## Authors

- Anthony Loukinas <<anthony.loukinas@redhat.com>>