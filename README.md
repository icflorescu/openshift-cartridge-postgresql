# Custom PostgreSQL cartridge for OpenShift

![PostgreSQL on OpenShift by Ionut-Cristian Florescu](https://cloud.githubusercontent.com/assets/581999/15590729/ce16d6f6-23a1-11e6-9b4d-41f8f17a32a3.png)

This is a custom OpenShift cartridge providing a contemporary PostgreSQL version, courtesy of [bigsql.org](http://www.bigsql.org/).

## Installation

**Warning! You're using this cartridge on your own responsibility**.

### Using the [web console](https://openshift.redhat.com/app/console/applications)

To install this cartridge in your existing OpenShift application, go to **"See the list of cartridges you can add"**, paste the URL below in **"Install your own cartridge"** textbox at the bottom of the page and click "Next".

    https://raw.githubusercontent.com/icflorescu/openshift-cartridge-postgresql/master/metadata/manifest.yml

### Using the command line

To add this cartridge to an existing application named `awesomeapp`, run:

    rhc cartridge add -a awesomeapp \
      https://raw.githubusercontent.com/icflorescu/openshift-cartridge-postgresql/master/metadata/manifest.yml

Using `rhc` utility, you can also create "from scratch" an applocation based on multiple cartridges. For instance, assuming you'd want a primary Node.js web application provided by [this community cartridge](https://hub.openshift.com/quickstarts/243-node-js-latest), run:

    rhc app create awesomeapp \
      https://raw.githubusercontent.com/icflorescu/openshift-cartridge-nodejs/master/metadata/manifest.yml \
      https://raw.githubusercontent.com/icflorescu/openshift-cartridge-postgresql/master/metadata/manifest.yml

## Usage

### Connect from a web application

Use the following environment variables to connect from an application running in the main web cartridge:
- `OPENSHIFT_PG_HOST`
- `OPENSHIFT_PG_PORT`
- `OPENSHIFT_PG_DATABASE`
- `OPENSHIFT_PG_USERNAME`
- `OPENSHIFT_PG_PASSWORD`

For instance, here's how you'd do it in a Node.js application using Knex.js:

    var knex = require('knex')({
      client: 'pg',
      connection: {
        host     : process.env.OPENSHIFT_PG_HOST,
        port:    : process.env.OPENSHIFT_PG_PORT,
        database : process.env.OPENSHIFT_PG_DATABASE,
        user     : process.env.OPENSHIFT_PG_USERNAME',
        password : process.env.OPENSHIFT_PG_PASSWORD
      }
    });

### Connect from a local development environment

Use `rhc port-forward` to connect from a client application running on your local development machine.

If your PostgreSQL instance is running in a separate gear, you can find its gear id by running:

    rhc app show application-name --gears

Then you can do port forwarding like this:

    rhc port-forward application-name -g gear-id

Connect your client application to the created proxy host and port.

Please **don't open issues in this repository to ask for help on using `rhc port-forward` or `rhc` utility in general**. Refer to the [OpenShift Online Documentation](https://developers.openshift.com/en/managing-port-forwarding.html) instead. If you still can't find what you're looking for, Google for it or ask on Stackoverflow.

## Notes

### Files and folder locations

- Upon installation, the [DevOps linux64 sandbox](http://www.bigsql.org/postgresql/installers.jsp) is downloaded from [bigsql.org](http://www.bigsql.org/) and expanded automatically in `${OPENSHIFT_DATA_DIR}.bigsql`;
- The PostgreSQL data folder is `${OPENSHIFT_DATA_DIR}.pgdata`;
- The configuration files `pg_hba.conf` and `postgresql.conf` are stored in the PostgreSQL data folder;
- Log files are created in `${OPENSHIFT_LOG_DIR}` folder;
- `psql` history files are created in `${OPENSHIFT_DATA_DIR}` folder;
- The PostgreSQL shared folder is `${OPENSHIFT_DATA_DIR}.bigsql/pg95/share/postgresql`;

### Superuser account

The cartridge installation script also creates a PostgreSQL superuser and saves its name and password in `OPENSHIFT_PG_ADMIN_USERNAME` and `OPENSHIFT_PG_ADMIN_PASSWORD` environment variables. You can use the superuser account to perform various administrative tasks (such as `CREATE EXTENSION ...`, `VACUUM ...`, etc.).

### Unsupported features

The autovacuum and statistics collection features are not supported by OpenShift setups (more info available [here](http://stackoverflow.com/questions/23215429/autovacuum-is-not-running-on-openshift-online-postgres-cartridge) and [here](https://bugzilla.redhat.com/show_bug.cgi?id=849428)).
A `vacuumdb -a -f` command is issued when executing the cartridge `tidy` script, so you can periodically use `rhc app tidy` to regain storage space.   

### Optional features

In order to conserve space, the `postgis` extension is not installed by default. In order to use it, you'll have to login to your cartridge and run `postgres/bin/install-postgis`, then issue `CREATE EXTENSION postgis` SQL statement in a privileged connection (see the note on the superuser account above).

### Full-text search for languages other than English

You can alter the default full-text search behavior by changing/adding the necessary files in the PostgreSQL shared folder (see the note on files and folder locations above). The files distributed by default in the PostgreSQL package are insufficient for languages other than English, but you can add your own `*.stop` or dictionary files, or even modify the standard `unaccent.rules` as needed.

## Read carefully before raising issues

I'm getting lots of questions from people just learning to do web development or simply looking to solve a very specific problem they're dealing with. While I will answer some of them for the benefit of the community, please understand that open-source is a shared effort and it's definitely not about piggybacking on other people's work. On places like GitHub, that means raising issues is encouraged, but **coming up with useful pull-requests is a lot better**. If I'm willing to share some of my code for free, I'm doing it for a number of reasons: my own intellectual challenges, pride, arrogance, stubbornness to believe I'm bringing a contribution to common progress and freedom, etc. Your particular well-being is probably not one of those reasons. I'm not in the business of providing free consultancy, so if you need my help to solve your specific problem, there's a fee for that.

## Asking for help or a new feature

See the note above. If you need help and are willing to pay for it, drop me a message. If you have an idea about a new feature that doesn't break existing ones and you're willing to invest effort to make it happen, have a look at the code and feel free to make a pull-request.

## Credits

See contributors [here](https://github.com/icflorescu/openshift-cartridge-postgresql/graphs/contributors).

If you find this repo useful, don't hesitate to give it a star and [spread the word](http://twitter.com/share?text=Checkout%20this%20custom%20PostgreSQL%20cartridge%20for%20OpenShift!&amp;url=http%3A%2F%2Fgithub.com/icflorescu/openshift-cartridge-postgresql&amp;hashtags=PostgreSQL,database,OpenShift&amp;via=icflorescu).

## License

The [ISC License](http://github.com/icflorescu/openshift-cartridge-postgresql/blob/master/LICENSE).
