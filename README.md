## Intro
Before setting up a Magento 2 development environment with Docker, make sure that the only running tool or dev software on your local is Docker. As it will have conflicts on the ports with other environment like Homebrew, XAMPP, etc.

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

# You should have already a copy of Magento 2 env.php and DB sql file */
# Syntax: install-script {project_name} {domain_name} {bitbucket_user} {php_version} {git_branch} {env_file} {db_file}
~/sites/docker-magento/install-script crabtree highspiritsliquor.test nico-atf 7.4 Dev ~/Downloads/ATF/Crabtree/env.php ~/Downloads/ATF/Crabtree/crabtree.sql
```

#### Sample Project Total Execution Time
<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/sample-project-ddi.png" alt="Sample Project DDI">

<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/sample-project-crabtree.png" alt="Sample Project Crabtree">

#### Setting up Magento 2 Multi-store Instance
To setup multi-store instances. For example, crabtree project has 4 stores/URLs.

These are the three files need to be configured.
- `docker-compose.dev.yml`
```bash
# Rename first file path of nginx.conf.sample to nginx.conf
services:
  app:
    volumes: &appvolumes
      ## Host mounts with performance penalty, only put what is necessary here
      - ./src/app/code:/var/www/html/app/code:cached
      - ./src/app/design:/var/www/html/app/design:cached
      - ./src/app/etc:/var/www/html/app/etc:cached
      - ./src/composer.json:/var/www/html/composer.json:cached
      - ./src/composer.lock:/var/www/html/composer.lock:cached
      - ./src/grunt-config.json.sample:/var/www/html/grunt-config.json:cached
      - ./src/Gruntfile.js.sample:/var/www/html/Gruntfile.js:cached
      - ./src/nginx.conf:/var/www/html/nginx.conf:cached
      - ./src/package.json.sample:/var/www/html/package.json:cached
```

- `images/nginx/conf/default.conf`: Update this file depending on the project configuration.

Before
<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/default.conf.png" alt="default.conf">

After
<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/crabtree-default.conf.png" alt="crabtree-default.conf">

- `src/nginx.conf`: Last, add these lines on nginx.conf
```bash
# START - Multisite customization
fastcgi_param MAGE_RUN_TYPE $MAGE_RUN_TYPE;
fastcgi_param MAGE_RUN_CODE $MAGE_RUN_CODE;
# END - Multisite customization
```

<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/project-nginx.conf.png" alt="nginx.conf">

After updating the files, register SSL to added stores/URLs.
```bash
bin/setup-ssl highspiritsliquor.test knb.test crabtreebrand.test chocmo.test
```

Then restart container to apply the changes.
```bash
bin/restart
```

If you have problem on configuring multi-store instances, you can check this video on what have you missed.
<a href="https://courses.m.academy/courses/set-up-magento-2-development-environment-docker/lectures/14780970" target="_blank">Configure multi-store instances in Docker with Nginx</a>

## Misc Info
Feel free to customize the automated script.
You can check here how the script works.
https://github.com/atfnico/magento2-docker/blob/master/install-script

## Credits

### Mark Shust's Docker Magento

I have referenced Mark Shust’s Docker Configuration for creating the automated script for installing Magento projects.
https://github.com/markshust/docker-magento

You can visit and review other functionalities, like the <a href="https://github.com/markshust/docker-magento#custom-cli-commands" target="_blank">Custom CLI Commands</a>.