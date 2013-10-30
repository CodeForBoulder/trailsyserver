# TrailsyServer

Trailsyserver is one of the components, along with [Trailsy](http://www.github.com/danavery/trailsy), of [To The Trails](http://tothetrails.com), a web application for storing and displaying hiking/biking/equestrian trail data.

To The Trails was developed by [2013 Code for America Fellows](http://www.codeforamerica.org/cities/summitcounty/) working with park and trail organizations in Summit County, Ohio.  The application is lightly customized for use in Summit County, but could be repurposed with minimal effort.

The Trailsyserver component includes:

  - a REST-style interface to trail data for consumption by [Trailsy](http://www.github.com/danavery/trailsy) (and possibly others). 
  - an interface for organizations that maintain trail data to upload and maintain their trail data in the Trailsy database, as well as provide status updates, closure updates, photos, and institutional information for invidivual trails.
  - data structure and API informed by [trail standards draft](https://docs.google.com/document/d/1frt5HkKTdqEaNEnfk2Dq9IYxctvPjVnoU_F33Px2zSQ).
 
## Requirements

  - Ruby 1.9.3 or newer. (All testing so far has been with Ruby 2.0.0)
  - An instance of Postgres with the PostGIS extension installed (can be local or remote--the current production version uses Heroku PostGIS hosting)
  - An Amazon S3 bucket for photo storage. (Not needed for development environment, which stores photos in the local filesystem. See "Local setup" below.)

We hope to merge this repo and the [Trailsy](http://www.github.com/danavery/trailsy) repo soon.

### Sample working resource requests
  - [http://trailsyserver-prod.herokuapp.com/trails.json]()
  - [http://trailsyserver-prod.herokuapp.com/trailheads.json?loc=41.1,-81.5]()
  - [http://trailsyserver-prod.herokuapp.com/trailsegments.json]()

## Setup notes

### Heroku setup

(Note: more instructions to come -- for example, you need a database user that can create databases and tables. After setup, you can remove those privileges if you prefer.)

To set up an instance on Heroku, you need a instance of PostGIS available. The production To The Trails application uses Heroku's PostGIS service.

An Amazon Web Services account is also required for photo storage in Amazon S3. The application uses [https://github.com/thoughtbot/paperclip](Paperclip) to manage photo attachments, so it could be converted to use any other Paperclip-supported storage backend.

To install on Heroku, you'll need to set the following app config vars:

 - BUILDPACK_URL:                         https://github.com/ddollar/heroku-buildpack-multi.git
 - GDAL_BINDIR:                           /app/vendor/gdal/1.10.0/bin
 - DATABASE_URL:                          postgis://[db_username]:[db_password]@[host]:[port]/[db_name]
 - TRAILSY_AWS_BUCKET:                    [your AWS S3 bucket name]
 - TRAILSY_AWS_ACCESS_KEY_ID:             [your AWS access key id]
 - TRAILSY_AWS_ACCESS_SECRET_ACCESS_KEY:  [your AWS access key]

To add these to your Heroku app (Heroku toolbelt required):

    heroku config:add BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
    heroku config:add GDAL_BINDIR=/app/vendor/gdal/1.10.0/bin
    heroku config:add DATABASE_URL=[your database URL]
    heroku config:add TRAILSY_AWS_BUCKET=[bucket] 
    heroku config:add TRAILSY_AWS_ACCESS_KEY_ID=[id] 
    heroky config:add TRAILSY_AWS_ACCESS_SECRET_ACCESS_KEY=[key]

To initialize the database:

    heroku run rake db:create && db:migrate && db:seed

To populate the database with sample trails, trailheads, and segments from Cuyahoga Valley National Park and Metro Parks, Serving Summit County:

    heroku run rake load:all

---

### Local setup

You need the GDAL/OGR package installed locally.

### Database environment variables required

There are two "development" Rails environments available: 

 - "development" uses a local instance of PostGIS, using the 'pg' gem adapter defaults, and local file storage for photos. You'll need to change the "development" portion of `config/database.yml` to point to your database.

 - "development-aws" uses a remote instance of PostGIS, and S3 for photos, based on the environmental variables below

    - TRAILSY_DB_DEV_USER=[name of database user]
    - TRAILSY_DB_DEV_HOST=[database host]
    - TRAILSY_DB_DEV_PASSWD=[database password]
    - TRAILSY_DB_DEV_DATABASE=[development database name]
    - TRAILSY_DEV_AWS_BUCKET=[your AWS S3 bucket name]
    - TRAILSY_DEV_AWS_ACCESS_KEY_ID=[your AWS access key id]
    - TRAILSY_DEV_AWS_SECRET_ACCESS_KEY=[your AWS access key]

 - "production" uses a remote instance of PostGIS, based on the environmental variables below (TODO: this isn't really true at the moment)

    - TRAILSY_DB_USER=[name of database user]
    - TRAILSY_DB_HOST=[database host]
    - TRAILSY_DB_PASSWORD=[database password]
    - TRAILSY_DB_DATABASE=[production database name]
    - TRAILSY_AWS_BUCKET=[your AWS S3 bucket name]
    - TRAILSY_AWS_ACCESS_KEY_ID=[your AWS access key id]
    - TRAILSY_AWS_SECRET_ACCESS_KEY=[your AWS access key]

### Other environment variables required

#### Path to GDAL binaries (for ogr2ogr)
 - GDAL_BINDIR=[path]  # depends on install -- if using OS X Homebrew, it's `/usr/local/bin`

#### Users to create during `rake db:seed`
Three users will be set up on DB creation. They should be email addresses. 

The first is the admin user. This user can approve new user accounts:

 - DEFAULT_ADMIN_USER

These users will have organization field values of "CVNP" and "MPSSC" respectively (more generic user names and organizations should be provide for testing, but for now they're not):

 - TEST_CVNP_USER
 - TEST_MPSSC_USER

All of these initial users will share the single password in the following environment variable:

 - DEFAULT_ADMIN_PASSWORD

Then run 

  rake db:create && db:migrate && db:seed

to initialize the database, and optionally run

  rake load:all

to load sample data from Cuyahoga Valley National Park and Metro Parks, Serving Summit County.

To start Trailsyserver, start it with `rails server`. If you're planning on using it with the [Trailsy](http://www.github.com/danavery/trailsy) fronr-end, change `API_HOST` in the first lines of `trailhead.js` to point to your instance of Trailsyserver. 
