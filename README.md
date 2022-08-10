## Intro
Before setting up a Magento 2 development environment with Docker, make sure that the only running tool or dev software on your local is Docker. As it will have conflicts on the ports with other environment like Homebrew, XAMPP, etc.

## Setup
#### Automated Setup (New Project)

```bash
# Create your project directory then go into it:
mkdir -p ~/sites/magento.test
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
# Skip this if you have already downloaded it before
cd ~/sites
git clone https://github.com/atfnico/magento2-docker.git

# Run ATF Docker automated script
# To see the syntax for help, run:
~/sites/magento2-docker/onelinesetup -h

# You should have already a copy of Magento 2 env.php and DB sql file.
# Those files can be downloaded from the Project's Development Environment Setup
# For example, https://abovethefray.atlassian.net/wiki/spaces/HSD/pages/2336620607/Development+Environment+Setup
# Syntax: onelinesetup {project_name} {domain_name} {bitbucket_user} {php_version} {git_branch} {env_file} {db_file}
~/sites/magento2-docker/onelinesetup crabtree highspiritsliquor.test nico-atf 7.4 Dev ~/Downloads/ATF/Crabtree/env.php ~/Downloads/ATF/Crabtree/crabtree.sql
```
While running the automated docker setup, you will be prompt to ask the user password.
<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/prompting-user-password.png" alt="prompting-user-password">

#### Sample Project Total Execution Time
<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/sample-project-ddi.png" alt="Sample Project DDI">

<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/sample-project-crabtree.png" alt="Sample Project Crabtree">

#### Manual Setup (Existing Project)
This is an alternative when you cannot set up your project using the onelinesetup.
Because sometimes we encountered a problem especially on composer installation.

```bash
# Run create vhost
~/sites/magento2-docker/create-vhost $DOMAIN

# After running create vhost, next will be setting up Magento instance

# Clone project repository
cd ~/sites/$DOMAIN;
git clone https://$BITBUCKET_USER@bitbucket.org/abovethefray/$PROJECT.git src;

# Verify git and switch to the branch you need to work
cd src;
git checkout develop;

# Then go back to project's docker directory
cd ../;

# Copy env file
mkdir -p src/app/etc/ && cp $ENV_FILE src/app/etc

# Make sure to configure project's nginx.conf
cp src/nginx.conf.sample src/nginx.conf

# Copy files downloaded from repo, host to container
bin/copytocontainer --all

# Trigger composer install to update latest package based 
# on the composer.json in the switched branch
bin/composer install -v

# Copying files from container to host after composer install
bin/copyfromcontainer --all

# Import the db to docker instance
bin/mysql < $DB_FILE

# Import app-specific environment settings
bin/clinotty bin/magento app:config:import

# Update elasticsearch configuration saved from the db - core_config_data
bin/mysql < ~/sites/magento2-docker/support-files/update-db.sql

# Run Magento CLI commands
bin/clinotty rm -rf generated/code/*
bin/clinotty bin/magento setup:upgrade
bin/clinotty bin/magento setup:di:compile
bin/clinotty bin/magento setup:static-content:deploy -f -j 10

# Re-indexing with Elasticsearch
bin/clinotty bin/magento indexer:reindex

# Update Magento base url
bin/clinotty bin/magento config:set web/secure/base_url https://"$DOMAIN"/
bin/clinotty bin/magento config:set web/unsecure/base_url https://"$DOMAIN"/

# Clearing the cache to apply updates
bin/clinotty bin/magento cache:flush

# Turning on developer mode
bin/clinotty bin/magento deploy:mode:set developer

mv .vscode src/

# Restart containers to apply changes
bin/restart
```

You can delete also a vhost site (including its images and containers) by running this command.
```bash
~/sites/magento2-docker/delete-vhost $DOMAIN
```

#### Setting up Magento 2 Multi-store Instance
To setup multi-store instances. For example, crabtree project has 4 stores/URLs.

These are the files need to be configured.

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

After updating the files, register SSL to added stores/URLs. Don't forget also to set up this in Magento URL configuration.
```bash
bin/setup-ssl highspiritsliquor.test knb.test crabtreebrand.test chocmo.test
```

Then stop, rebuild container and clean cache to make sure it will apply the changes.
```bash
bin/stop;
docker-compose build --no-cache --pull;
bin/cache-clean;
```

If you have problem on configuring multi-store instances, you can check this video on what have you missed.
<a href="https://courses.m.academy/courses/set-up-magento-2-development-environment-docker/lectures/14780970" target="_blank">Configure multi-store instances in Docker with Nginx</a>

#### Uploading pub media to Docker container
Copy the extracted pub media files to src/pub, then run below command to copy it to the container.
```bash
bin/copytocontainer pub/
```
For more info of managing files & folders within Docker containers, you can watch this video.
https://courses.m.academy/courses/set-up-magento-2-development-environment-docker/lectures/9064338

#### Accessing custom executable php in browser
If you want to access executable php in the browser, you can add it to src/pub.
<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/custom-pub-php.png" alt="custom-pub-php">
Configured it in src/nginx.conf.
<img src="https://raw.githubusercontent.com/atfnico/magento2-docker/master/docs/custom-pub-php-nginx-conf.png" alt="custom-pub-php-nginx-conf">
Then run, `bin/restart`.

## Misc Info
Feel free to customize the automated script.
You can check here how the script works.
https://github.com/atfnico/magento2-docker/blob/master/onelinesetup

### Check docker logs
```bash
bin/status
```
Check docker service status and get its name, then check the logs by running,
```bash
docker logs crabtree-phpfpm-1
```

## Development

#### Pushing code changes to repository
We can do this by accessing the src folder and do the git commands there.

For example,
```bash
cd src;
git status;
git add app/;
git commit -m "initial commint";
git push origin master;
```

#### Coding with Text Editor
I recommend to point the directory to src folder when using a text editor or IDE such as VS Code.

## Credits

### Mark Shust's Docker Magento

I have referenced Mark Shust’s Docker Configuration for creating the automated script for installing Magento projects.
https://github.com/markshust/docker-magento

You can visit and review other functionalities, like the <a href="https://github.com/markshust/docker-magento#custom-cli-commands" target="_blank">Custom CLI Commands</a>.

I recommend also to be familiarized on how Docker Magento works.
Visit this course by Mark Shust for more info, https://courses.m.academy/courses/set-up-magento-2-development-environment-docker/.