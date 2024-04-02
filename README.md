# Google Cloud SQL proxy buildpack

This heroku buildpack adds the Google Cloud SQL proxy to your app to enable
connections to SQL instances in the GCP.

## Prerequisite

- Google service account with the Cloud SQL/Cloud SQL client role
- JSON key for the service account
- Google Cloud SQL instance set up with a postgresql user able to connect
  to the database

## Install

Add the proxy to your buildpacks. It's important that this buildpack should
not be the last one in the list, as that's used by heroku to determine your
startup processes. (--index=1)

     heroku buildpacks:add --index=1 https://github.com/exostack/heroku-buildpack-cloud-sql-proxy

Add the GCP JSON credentials as `GSP_CREDENTIALS` env variable to you app. If you provide `GSP_CREDENTIALS_FILE_PATH`, the buildpack will read the credentials from the file at the given path.

Set the instance the proxy should connect to with the `GSP_INSTANCES` env
variable. Format: `<instance connection name>=tcp:<local port>`. You can get the
**instance connection name** from the Cloud SQL console overview.

Set the connection string for your DB library to
`postgres://<username>:<password>@localhost:<local port>/<database-name>`

> [!WARNING]
> You need to launch the proxy yourself, before your app tries to connect to the database.

The best place to do it is in the `.profile` file (launched by heroku before everything, see https://devcenter.heroku.com/articles/dynos#the-profile-file)

Start the proxy before your app tries to connect to the database by e.g. adding the following to your `.profile`:

```bash
# with the credentials as env variable
printf "%s" "$GSP_CREDENTIALS" > /app/google/credentials.json
exec /app/google/bin/cloud_sql_proxy --credentials-file=/app/google/credentials.json $GSP_INSTANCES &

# or with the credentials as file path
exec /app/google/bin/cloud_sql_proxy --credentials-file=$GSP_CREDENTIALS_FILE_PATH $GSP_INSTANCES &
```
