# IFDB

This is the source code for [IFDB](https://ifdb.org), the Interactive Fiction Database.

# Preparing Your Own Development Environment

The IFDB web app is a LAMP app (Linux, Apache, MySQL, PHP). The development environment runs on Docker.

1. Install Docker and Docker Compose. [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)

2. Run `./prepare_dev_environment.sh` in this directory. That will download the current latest IFDB database backup from IFArchive, and put it in a place where Docker will expect to find it.

3. Run `docker-compose up`. That will launch a MySQL Docker container and an Apache container with PHP, available on port 8080.

4. Go to `http://localhost:8080` on your machine. You should see IFDB running. You can login as `ifdbadmin` at `ifdb.org` with the password `secret` to sign in as an administrator. The administrator has access to the "system maintenance panel" at `http://localhost:8080/adminops`

5. Optionally, you can query the database using phpMyAdmin at `http://localhost:8081` or run `docker exec -it ifdb_db_1 mysql -psecret ifdb` to use the MySQL command-line interface.

# Database Changes

Changes to the web site should update immediately but changes to the database require a few more steps.

Make all database schema and content changes in `./sql/patch-schema.sql`. After saving your changes, type the following three commands while in this directory:

```
docker-compose down
./prepare_dev_environment.sh
docker-compose up
```

All of your database changes should now be available.

## Known Issues with the Development Environment

* Sending email doesn't work. That's unfortunate, because if you want to create a user, you'll need to login with an activation code. After you try to create a new user, you should be able to see the email text in the Docker logs. Search for `EMAIL: NOT SENDING EMAIL IN LOCAL DEVELOPMENT MODE`.
* Game box-art images load from the production IFDB site, not the dev environment. IFDB uses a separate "images" database that isn't part of the IFArchive backup. We'll need to generate a backup of that database and make it available on IFArchive, or, at the very least, provide a way for developers to download images from the real IFDB.

## Troubleshooting

To reset the Docker environment, stop `docker-compose` with Ctrl-C, then run `docker-compose down` to delete the images. If you're modifying the `Dockerfile`, you might also need to `docker rmi ifdb_web` to clean out state.

If you see a 500 error loading `http://localhost:8080`, check the Docker logs. If you see one of these errors, then something went wrong during the `./prepare_dev_environment.sh` step.

* `PHP Warning:  include_once(local-credentials.php): failed to open stream: No such file or directory`
* `PHP Warning:  include_once(): Failed opening 'local-credentials.php' for inclusion`
* `PHP Fatal error:  Uncaught Error: Call to undefined function localCredentials()`

If you see this, reset the Docker environment (see above), then run `./prepare_dev_environment.sh` and `docker-compose up` again.
