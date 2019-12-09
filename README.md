# docker-awx-tower

This repository is a basic docker-compose configuration that works for both AWX and Ansible Tower using the new quay Tower images. I'm in the process of working on a better more dynamic solution using Ansible to configure these environments, but I couldn't find a *working* example of Ansible Tower/AWX just using compose files in Docker.

Update: I've updated this repo to now be a bit more dynamic. You can configure multiple environments by running the `build.yml` and answering prompts. This will create new build directories with all the assets needed to run Tower/AWX in Docker.

## Usage

#### Configure tower-vars.yml

Create your own copy of `tower-vars-base.yml`.

```bash
cp install-vars-base.yml install-vars.yml
```

**NOTE: If you would prefer to not use the docker python modules, or are in an environment where you cannot use them, modify the following in your tower-vars.yml**

```yaml
# Use shell module instead of docker modules
use_docker_modules: no
```

By default, both in the role and in the tower-vars.yml you've just created we are specifiying an Ansible Tower install which requires a License. If you are looking to use the open-source free variant of Ansible Tower (Ansible AWX), then uncomment the AWX section, and comment out the Tower, otherwise continue with Tower.

```yaml
# Ansible Tower
# task_image: quay.io/ansible-tower/ansible-tower
# web_image: quay.io/ansible-tower/ansible-tower
# install_version: 3.6.1

# Ansible AWX
task_image: ansible/awx_task
web_image: ansible/awx_web
install_version: 9.0
```

- Latest AWX versions: https://hub.docker.com/r/ansible/awx_task/tags
- Latest Tower versions: https://quay.io/repository/ansible-tower/ansible-tower?tab=tags

Next, edit the variables in which define how your Tower/AWX instance is to be configured. This is optional, by default what's uncommented will work. Only uncomment and modify if you have specific reasoning.

```yaml
postgres_username: postgres
postgres_password: postgres

rabbitmq_username: awx
rabbitmq_password: awx

### Questions asked in playbook prompt
# Use these if you plan on removing the survey prompt

tower_username: admin
tower_password: password

tower_environment: dev
tower_http_port: 80
```

#### Start Building
Run the build.yml to start the docker-compose build process

```bash
ansible-playbook build.yml -e @install-vars.yml
```

This playbook will prompt you for the following:

- Tower Environment
- Tower Username
- Tower Password
- Tower HTTP Port

It will build two docker objects needed for your compose to work:

- Volume: "{{ tower_environment }}_db"
- Network: "{{ tower_environment }}_net"

#### Run your Tower/AWX!

```bash
# Change directory to your new build
cd build/{{ tower_environment }}/

# Run your Tower/AWX
docker-compose up -d
```

*Tip: If you run `docker-compose up` without the `-d` all logs will dump to your active terminal. Beware: If you CTRL+C it will tear down your Tower.. but don't worry your database will persist*


## Legacy Usage

I've moved the static Tower/AWX compose definitions into the `static/` subfolder. These are considered working out of the box configurations that offer little customization or re-useability from a new instance perspective. Use these when you need a single working Tower/AWX setup.

### AWX Static Setup (Legacy)

```bash
# Create AWX Database Volume
docker volume create awx_web

# Change to AWX directory
cd static/awx

# Initialize the AWX application in the background denoted by -d
docker-compose -f awx-compose.yml up -d
```

### Tower Static Setup (Legacy)

```bash
# Create Tower Database Volume
docker volume create tower_web

# Change to Tower directory
cd static/tower

# Initialize the Tower application in the background denoted by -d
docker-compose -f tower-compose.yml up -d
```

### Database/Data Removal (Legacy)

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
