# Deployment Instructions

![Docker](https://img.shields.io/badge/Docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1.svg?style=for-the-badge&logo=mysql&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-%23777BB4.svg?style=for-the-badge&logo=php&logoColor=white)
![Laravel](https://img.shields.io/badge/Laravel-%23FF2D20.svg?style=for-the-badge&logo=laravel&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-%236DA55F.svg?style=for-the-badge&logo=node.js&logoColor=white)
![Vue.js](https://img.shields.io/badge/Vue.js-%2335495e.svg?style=for-the-badge&logo=vuedotjs&logoColor=%234FC08D)
![Nginx](https://img.shields.io/badge/Nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)

## Description
This is a starter kit of Docker services for developing and running a REST API application. The structure includes predefined configurations, and all development essentially takes place in two folders: ``backend`` and ``frontend``.

## Requirements
- Linux or WSL2 (the Windows file system significantly slows down Docker).
- Docker (if you use Docker Desktop on Windows, check compatibility with WSL).

## Using the repository
Since this is not a full-fledged application but just a helper starter package, you should download and unzip it as a regular archive rather than clone it.
```bash
curl -L https://github.com/iXorr/laravel-vue-sample/archive/refs/heads/pure-sample.tar.gz \
| tar -xz --strip-components=1
```

And inside the ``backend`` and ``frontend`` folders, either initialize new repositories or use existing ones.

## Service versions
Adjust the version of a specific service in the corresponding Dockerfile (for example, if you change the PHP version, you need to go to ``services/php/Dockerfile``). Also, remember that framework versions depend on the service version: for example, for PHP 8.1, you need to install Laravel 10.x.

## Web server configuration
All requests starting with ``/api/`` are redirected to the Laravel application in the ``backend/public`` directory. All other requests are intercepted through a proxy and redirected to the frontend, so if the frontend isn't running, Nginx will throw an error.

## Redefining access ports

If you *have* and *already* running services that use ports ``80``, ``3306``, or ``5173``, you can redefine the access ports specifically for this starter kit.
Copy the ``.env`` file based on ``.env.example`` to the root of your project and change the environment variables there to the free ports you need.

Don't forget to take port changes into account when developing your application (for example, access the server via ``localhost:180`` instead of ``localhost``)!

```bash
cp .env.example .env
```

## Overriding Service Names

Container names can conflict with each other (even though projects are launched in different directories and Docker Desktop separates them, container names are considered GLOBAL), so the environment file contains the variable ``APP_ID``. It adds a prefix to each newly created service.

## Getting Started
1. This version of the starter package doesn't come with any presets. So, to get started, just create a project.
   ```bash
   docker compose build
   ```

2. Remove the .gitkeep files from the ``backend`` and ``frontend`` folders. Then download the framework versions you need.
   ```bash
   docker compose run --rm backend composer create-project laravel/laravel:^10 .
   docker compose run --rm frontend npm create vite .
   ```

   Or clone repositories of existing projects into them.
   ```
   git clone <PROJECT-LINK> backend
   git clone <PROJECT-LINK> frontend
   ```

   Make sure all dependencies are installed. Otherwise, run these commands.
   ```
   docker compose run --rm backend composer install
   docker compose run --rm frontend npm install
   ```

3. You can also add your own dependencies: for both the frontend and backend.
   ```bash
   docker compose run --rm frontend npm install --dev vue-router
   ```

4. Start the services.
   ```bash
   docker compose up -d
   ```

5. Once all services are running, prepare the backend for production.

   You need to set up the database environment for Laravel. In the ``backend`` folder, copy the ``.env`` file from ``.env.example`` and make the following changes.
   ```
   DB_CONNECTION=mysql
   DB_HOST=${DB_HOST} # docker container name
   DB_DATABASE=app_db # docker db conf
   DB_USERNAME=admin # docker db conf
   DB_PASSWORD=admin # docker db conf
   ```

   Also, pay attention to the session driver: since version 11, Laravel uses the database by default, but there is no migration for the corresponding table.

   Make sure the ``APP_KEY`` has been generated and migrations have been run. Otherwise, use these commands.
   ```bash
   docker compose run --rm backend php artisan key:generate
   docker compose run --rm backend php artisan migrate --seed
   ```

   Finally, grant the system permissions to work with the folders where media files or cache are stored.
   ```bash
   docker compose run --rm backend chown -R www-data:www-data storage bootstrap/cache
   docker compose run --rm backend chmod -R 775 storage bootstrap/cache
   ```

6. If you don't have permissions to any of the installed folders (which is often the case on Linux systems), change their ownership to yourself.
   ```
   sudo chown -R <YOUR-USER>:<YOUR-USER-GROUP> ./frontend/*
   ```

   But be careful with the ``backend/bootstrap/cache`` and ``backend/storage`` folders: their owner must be ``www-data``! This is the user group through which the web server and PHP interpreter access the Laravel application.