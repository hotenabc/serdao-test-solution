# serdao-test-solution
1. Setting Up Symfony Project
First, make sure we have Symfony CLI installed on our system.
To create a new Symfony project, run the following command:

bash
symfony new serdao_symfony_app --full

This will create a new Symfony project with all the standard components included.

2. Defining Dependencies and Configuration
Next, navigate to the newly created project directory:

bash
cd serdao_symfony_app

In a Symfony application, dependencies are managed using Composer, so we need to ensure that the composer.json file is present and includes all required dependencies. Once we’ve added or modified the dependencies, run:

bash
composer install

Now, configure our Symfony application as per our specific requirements, making any necessary adjustments to the database connection, caching, and other services.

3. Creating a Dockerfile for Symfony
A Dockerfile is a text document that contains all the commands required to assemble a Docker image. To containerize our Symfony application, we need to create a Dockerfile.

4. Base Image Selection
The first step is to select a base image for our Symfony application. We’ll use an official PHP base image, which is available on Docker Hub:

Dockerfile
FROM php:8.0-fpm

This base image includes PHP and FPM (FastCGI Process Manager) to serve PHP applications efficiently.

5. Copying Symfony Files
Next, we need to copy our Symfony application files into the Docker image. We’ll use the COPY command to achieve this:

Dockerfile
WORKDIR /var/www/html
COPY . /var/www/html

This sets the working directory inside the container to /var/www/html and copies all the files from our current directory (Symfony project directory) into the container’s working directory.

6. Installing Dependencies
To run a Symfony application, we need to install its dependencies. We can achieve this by executing Composer inside the container:

Dockerfile
RUN apt-get update \
    && apt-get install -y libzip-dev zip \
    && docker-php-ext-configure zip --with-libzip \
    && docker-php-ext-install zip pdo pdo_mysql
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer install

Here, we install the required system dependencies, enable the PHP zip extension, and install the PHP extensions needed to run Symfony. Additionally, we install Composer and execute composer install to install the application-specific dependencies.

7. Exposing the Application
Finally, we need to expose the PHP-FPM port, allowing us to access the Symfony application from the host:

Dockerfile
EXPOSE 9000

This command informs Docker that the container will listen on port 9000, which is the default port for PHP-FPM.

8. Building the Docker Image
With the Dockerfile ready, we can now build the Docker image for our Symfony application.

9. Building the Image
To build the Docker image, execute the following command in the same directory as your Dockerfile:

bash
docker build -t serdao_symfony_app .

Here, -t is used to tag the image with a name (serdao_symfony_app in this case).

10. Verifying the Image
Once the build process completes, we can verify that the image is created successfully by running:

bash
docker images

We should see the newly built image in the list.

11. Compose File for Symfony
Create a docker-compose.yml file in our Symfony project’s root directory and define the services required for our Symfony application:

yaml
version: '3.8'

services:
  php:
    build: .
    container_name: serdao_symfony_app
    volumes:
      - .:/var/www/html
    ports:
      - "9000:9000"
    depends_on:
      - db
    environment:
      APP_ENV: dev
      SYMFONY_DOTENV_VARS: APP_ENV
    networks:
      - symfony_net

  db:
    image: mysql:5.7
    container_name: serdao_symfony_db
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: symfony_db
      MYSQL_USER: symfony_user
      MYSQL_PASSWORD: symfony_password
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - symfony_net

networks:
  symfony_net:
    driver: bridge

volumes:
  db_data:

12. Configuring Symfony for Database Connection
Next, we need to update Symfony’s database configuration to use the database container. In the config/packages/doctrine.yaml file, configure the database connection as follows:

yaml
doctrine:
    dbal:
        driver: pdo_mysql
        url: '%env(resolve:DATABASE_URL)%'
By using the %env(resolve:DATABASE_URL)% placeholder, Symfony will fetch the value of the DATABASE_URL environment variable from the container’s environment.

With this configuration, Symfony should now be able to connect to the MySQL database running in the separate db container.
