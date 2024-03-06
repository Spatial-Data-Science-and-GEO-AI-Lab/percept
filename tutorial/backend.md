# Setting up the backend server


## Creating the databases

You should have PostgreSQL (v13 or higher) and a suitable PostGIS package installed, as described in the Dependencies section. In most distributions it is easiest to create the databases and enable extensions as the `postgres` special user. For example, if you are logged in as your normal user (as recommended earlier in this tutorial) and you wish to create the three recommended databases with our recommended names, then run the following commands:

    sudo -u postgres createdb -O $USER percept-test
    sudo -u postgres psql -c 'CREATE EXTENSION postgis;' percept-test
    sudo -u postgres createdb -O $USER percept-dev
    sudo -u postgres psql -c 'CREATE EXTENSION postgis;' percept-dev
    sudo -u postgres createdb -O $USER percept-prod
    sudo -u postgres psql -c 'CREATE EXTENSION postgis;' percept-prod

You will now have databases named `percept-test`, `percept-dev` and `percept-prod`, all of which are now owned (the `-O` option) by your normal user (provided by the `$USER` environment variable if you are logged in).

## Getting the code

You should have NodeJS installed as described in the Dependencies section. Obtain a copy of the `percept-backend` source code from GitHub and put it into any convenient directory. For example, `cd $HOME && git clone https://github.com/Spatial-Data-Science-and-GEO-AI-Lab/percept-backend && cd percept-backend`.

You should now be in the `percept-backend` directory.

## The 3 databases

We ask you to create three databases for the following purposes:
- `percept-test`: a scratch database that will be frequently deleted and rebuilt whenever you run the test suite
- `percept-dev`: for testing the deployment and development (if you get that far)`
- `percept-prod`: the real database for 'production purposes'

If you change the names, that is fine, but try to be aware of the different purposes of these databases and be sure to adjust the configuration files appropriately.

## Configuring the backend

Copy the sample configuration to `config.js` using the command: `cp config.js.sample config.js`

The contents of the configuration look like this:

    // Name of the PostgreSQL database to use:
    const dbname = 'percept-dev';
    // Hostname (for TCP connections) or directory (for UNIX socket) of the database:
    const dbhost = '/var/run/postgresql';
    // Name of the PostgreSQL database to use for 'npm test' testsuite purposes only:
    const testdbname = 'percept-test';
    // Hostname (for TCP connections) or directory (for UNIX socket) of the testing database:
    const testdbhost = '/var/run/postgresql';


    // END OF CONFIGURATION

    // Export configuration:
    module.exports = {dbname, dbhost, testdbname, testdbhost, public_url};


The sample configuration will work for now if you have chosen our default database names. Otherwise you will need to adjust the names. For now, we will use the `percept-dev` database as the `dbname`, but later when you are ready to deploy you will come back and change this to `percept-prod` (or equivalent).

## Installing the software

In the project directory, run: `npm install`

This should downloand and install all the NodeJS dependencies.

## Seeding the databases

The databases are empty at this point and we need some data to populate them. That data is stored in `seeds` or `seed files`. For testing purposes, we can simply use a sample of data. Later, when we build the production database, we will want to be very specific about what data (e.g. imagery) is in the seed files.

The `<sqldir>` that you created in the processing stage should have numerous files ending with `_insert.sql`. These can now be used to seed the database(s). For example, starting with the `percept-dev` database for testing our deployment, we can assemble all of the `INSERT` statements into a seed file, e.g. using the following command:

    find <sqldir> -name '*_insert.sql' -exec cat \{\} \; >> seeds/dev/01_inserts.sql


