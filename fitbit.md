# radar fitbit

Needs optional services, `radar-fitbit-connector`, `radar-rest-sources-backend`, probably `radar-rest-sources-authorizer`

```
docker-compose -f docker-compose-lite.yml -f optional-services-lite.yml up -d radar-fitbit-connector radar-rest-sources-backend radar-rest-sources-authorizer
```

NB config...
radar-fitbit-connector:
- etc/fitbit/docker/source-fitbit.properties
- etc/fitbit/docker/users
- (fitbit-logs)

radar-rest-sources-backend:
- etc/rest-source-authorizer/
- specifically app-includes/rest_source_clients_configs.yml ?

Ahh, see
```
For the Fitbit Connector, please specify the FITBIT_API_CLIENT_ID and FITBIT_API_CLIENT_SECRET in the .env file. Then copy the etc/fitbit/docker/users/fitbit-user.yml.template to etc/fitbit/docker/users/fitbit-user.yml and fill out all the details of the fitbit user.
```
and [more info](https://github.com/RADAR-base/RADAR-REST-Connector#usage)

Client id and secret end up in etc/fitbit/docker/source-fitbit.properties.
Old-style [fitbit console](https://dev.fitbit.com/apps)

For this file-style I think you need to set 
`fitbit.user.repository.class` to `org.radarbase.connect.rest.fitbit.user.YamlUserRepository`
and leave `fitbit.user.dir` as default (`/var/lib/kafka-connect-fitbit-source/users`)

Generate user id, access & refresh tokens using 
[fitbit page](https://dev.fitbit.com/apps/oauthinteractivetutorial),
"authorization code flow", working through the steps.

Add an entry to etc/fitbit/docker/users/fitbit-user.yml
Be careful of dates or it generates a LOT of requests - exeeds quotas.

(kafka lost all the topics - installed again)



Note, if using the radar-rest-sources-backend...

That also has the fitbit app client id/secret, in etc/rest-source-authorizer/rest_source_clients_configs.yml

So add those...
not much happening here... (what users?)
