# pcf-c2c-networking

## Download the artifacts

```bash
wget https://github.com/pivotal-education/pcf-articulate-code/releases/download/0.0.1/articulate-0.0.1-SNAPSHOT.jar
wget https://github.com/pivotal-education/pcf-attendee-service-code/releases/download/0.0.1/attendee-service-0.0.1-SNAPSHOT.jar
```

# Set some vars

```bash
INITIALS=<YOUR_INITIALS>
EXTERNAL_DOMAIN=<YOUR_DOMAIN> # e.g. apps.cfapps.io
```

# Push the apps with external routes

```bash
cf push articulate -p articulate-0.0.1-SNAPSHOT.jar -n articulate-${INITIALS} --no-start
cf push attendee-service -p attendee-service-0.0.1-SNAPSHOT.jar -n attendee-service-${INITIALS} --no-start
```

## Create and bind the services

```bash
cf create-service p.mysql db-small mydata
cf bind-service attendee-service mydata

cf create-user-provided-service attendee-service-ups -p uri <<< "http://attendee-service-${INITIALS}.${EXTERNAL_DOMAIN}/attendees
cf bind-service articulate attendee-service-ups
```
## Start the apps

```bash
cf start attendee-service
cf start articulate
```
