# docker-awx-tower

This repository is a basic docker-compose configuration that works for both AWX and Ansible Tower using the new quay Tower images. I had to split the two configurations into their own directories due to issues with docker-compose overwriting the variations. I'm in the process of working on a better more dynamic solution using Ansible to configure these environments, but I couldn't find a *working* example of Ansible Tower/AWX just using compose files in Docker.

## Usage

For the AWX/Tower web container definitions in each respective compose file, you may want to modify the exposed HTTP port from the default "80", especially if you're planning on running multiple instances.

To run multiple instances, I would for now copy the tower/awx directories into new directories, and modify the compose yaml accordingly ensuring you use different HTTP ports as well as container_names.

```bash
# Create a new tower copy
cp -r tower/ tower-dev/

# Edit container_name: and ports: "80:8052"
vi tower-dev/tower-compose.yml

# Launch new Tower environment
cd tower-dev/
docker-compose -f tower-compose.yml up -d
```

### AWX Setup

```bash
# Create AWX Database Volume
docker volume create awx_web

# Change to AWX directory
cd awx

# Initialize the AWX application in the background denoted by -d
docker-compose -f awx-compose.yml up -d
```

### Tower Setup

```bash
# Create Tower Database Volume
docker volume create tower_web

# Change to Tower directory
cd tower

# Initialize the Tower application in the background denoted by -d
docker-compose -f tower-compose.yml up -d
```

### Database/Data Removal

Since we created the postgres SQL data container volume by hand, we will need to remove the volume by hand if we want to "reset" our Tower instance.

```bash
# Remove Tower Database Volume
docker volume rm tower_db

# Remove AWX Database Volume
docker volume rm awx_db
```

The reason we force you to create these volumes by hand in the first place is so that you don't accidently delete or lose track of the volume in Docker. (This keeps your compose working for the long haul).

## Authors

- Anthony Loukinas <<anthony.loukinas@redhat.com>>
