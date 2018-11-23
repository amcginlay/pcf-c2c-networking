# pcf-c2c-networking

## Download the artifacts

```bash
wget https://github.com/pivotal-education/pcf-articulate-code/releases/download/0.0.1/articulate-0.0.1-SNAPSHOT.jar
wget https://github.com/pivotal-education/pcf-attendee-service-code/releases/download/0.0.1/attendee-service-0.0.1-SNAPSHOT.jar
```

## Set some vars

```bash
INITIALS=<YOUR_INITIALS>
DOMAIN=<YOUR_DOMAIN> # e.g. cfapps.io
```

## Push all the apps with external routes

This is the "old school" way of deploying apps which need to talk to each other.

```bash
cf push articulate -p articulate-0.0.1-SNAPSHOT.jar -n articulate-${INITIALS} -d ${DOMAIN} --no-start
cf push attendee-service -p attendee-service-0.0.1-SNAPSHOT.jar -n attendee-service-${INITIALS} -d ${DOMAIN} --no-start
```

## Create and bind the services

```bash
cf create-service p-mysql 100mb mydata # ... or similar
cf bind-service attendee-service mydata

cf create-user-provided-service attendee-service-ups -p uri <<< "http://attendee-service-${INITIALS}.${DOMAIN}/attendees"
cf bind-service articulate attendee-service-ups
```
## Start the apps

```bash
cf start attendee-service
cf start articulate
```

Data entry is possible _but_ the routes to both apps should be navigable from a browser

## Internalize the attendee-service route

Replace the external route with an internal one

```bash
cf map-route attendee-service apps.internal -n attendee-service-${INITIALS}
cf unmap-route attendee-service ${DOMAIN} -n attendee-service-${INITIALS}
```

## Enable the apps to communicate internally

```bash
cf add-network-policy articulate --destination-app attendee-service
```

## Update the service endpoint configuration

```bash
cf update-user-provided-service attendee-service-ups -p uri <<< "http://attendee-service-${INITIALS}.apps.internal:8080/attendees"
cf restart articulate
```

Data entry is still possible but the `attendee-service` route will no longer be navigable from a browser.
Note the internal port 8080 is bound straight through the from the container without colliding in the host.

This aludes to the presence of software defined networking.
It's also why each of our containers internally runs an Envoy process.
Try running `cf ssh articulate -c "ps -ef | grep envoy"`.
