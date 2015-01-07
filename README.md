# PiSQL

A base Docker image to run a MySQL database server on the Raspberry Pi.

## Usage

Create the image by running the following command from the PiSQL directory:

```docker build -t prinsmike/pisql .```

Then run a container and bind to port 3306:

```docker run -d -p 3306:3306 prinsmike/pisql```

The first time you run the container, a new `admin` user will be created with all MySQL privileges, and a randomly generated password. The password can be obtained from the docker logs:

```docker logs <CONTAINER_ID>```

The output will appear like the following:

        ========================================================================
        You can now connect to this MySQL Server using:

            mysql -uadmin -p47nnf4FweaKu -h<host> -P<port>

        Please remember to change the above password as soon as possible!
        MySQL user 'root' has no password but only allows local connections
        ========================================================================

In this case, `47nnf4FweaKu` is the password allocated to the `admin` user.

Remember that the `root` user has no password but it's only accessible from within the container.

## Setting a specific password for the admin account

If you want to set your own password instead of having one randomly generated, you can set the environment variable `MYSQL_PASS` when running the container:

```docker run -d -p 3306:3306 -e MYSQL_PASS="mysecretpassword" prinsmike/pisql```

The admin username can also be set via the `MYSQL_USER` environment variable.

## Mounting the database file volume

In order to persist the database data, you can mount a local folder from the host on the container to store the database files. To do so:

```docker run -d -v /path/in/host:/var/lib/mysql prinsmike/pisql /bin/bash -c "/usr/bin/mysql_install_db"```

This will mount the local folder `path/in/host` inside the docker container in `/var/lib/mysql`, which is where MySQL will store the database files by default. `mysql_install_db` creates the initial database structure.

Note that your host must have `/path/in/host` available when you run your docker image.

After this you can start your mysql image, but this time using `/path/in/host` as the database folder:

```docker run -d -p 3306:3306 -v /path/in/host:/var/lib/mysql prinsmike/pisql```

## Mounting the database file volume from other containers

Another way to persist the database data is to store database files in another container. To do so, first create a container that will hold the database files:

```docker run -d -v /var/lib/mysql --name db_vol -p 22:22 resin/rpi-raspbian```

This will create a new ssh-enabled container and use its folder `/var/lib/mysql` to store MySQL database files. You can specify any name fro the container by using the `--name` options, which will be used in the next step.

After this you can start your MySQL image using volumes in the container created above (put the name of the container in `--volumes-from`)

```docker run -d --volumes-from db_vol -p 3306:3306 prinsmike/pisql```

## Thanks

This is based on the work by tutumcloud at http://www.github.com/tutumcloud/tutum-docker-mysql.
