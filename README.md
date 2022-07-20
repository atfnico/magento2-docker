## Setup
#### Automated Setup (New Project)

```bash
# Create your project directory then go into it:
mkdir -p ~/Sites/magento
cd $_

# Download the Docker Compose template:
curl -s https://raw.githubusercontent.com/markshust/docker-magento/master/lib/template | bash

# Download the version of Magento you want to use with:
bin/download 2.4.4

# or for Magento core development:
# docker-compose -f docker-compose.yml up -d
# bin/setup-composer-auth
# bin/cli git clone git@github.com:magento/magento2.git .
# bin/cli git checkout 2.4-develop
# bin/composer install

# Run the setup installer for Magento:
bin/setup magento.test

open https://magento.test
```

#### Automated Setup (Existing Project)

```bash
# Download the magento2-docker from the below git repo 
# https://github.com/atfnico/magento2-docker
cd ~/sites
git clone https://github.com/atfnico/magento2-docker.git

# Run ATF Docker automated script
# To see the syntax for help, run:
~/sites/docker-magento/install-script -h

# Syntax: install-script {project_name} {domain_name} {bitbucket_user} {php_version} {git_branch} {env_file} {db_file}
~/sites/docker-magento/install-script crabtree highspiritsliquor.test nico-atf 7.4 Dev ~/Downloads/ATF/Crabtree/env.php ~/Downloads/ATF/Crabtree/crabtree.sql
```

#### Sample Project Total Execution Time
<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/sample-project-ddi.png" alt="Sample Project DDI">

<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/sample-project-crabtree.png" alt="Sample Project Crabtree">

## Misc Info
Feel free to customize the automated script.
You can check here how the script works.
https://github.com/atfnico/magento2-docker/blob/master/install-script

## Credits

### Mark Shust's Docker Magento

I have referenced Mark Shust’s Docker Configuration for creating the automated script for installing Magento projects.
https://github.com/markshust/docker-magento

You can visit and review other functionalities, like the <a href="https://github.com/markshust/docker-magento#custom-cli-commands" target="_blank">Custom CLI Commands</a>.