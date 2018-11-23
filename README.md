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

Data entry is possible in `articulate` via the Services tab _but_ routes to
both of the apps are navigable from a browser

## Internalize a route

We don't want the outside world directly accessing the `attendee-service` so
replace its route with one that is only accessible internally.

```bash
cf unmap-route attendee-service ${DOMAIN} -n attendee-service-${INITIALS}
cf map-route attendee-service apps.internal -n attendee-service-${INITIALS}
```

## Enable internal communication between the front-end and back-end

Internal communication between apps can only happen if we say so

```bash
cf add-network-policy articulate --destination-app attendee-service
```

## Update the service endpoint configuration

A reference to the old endpoint was wired into VCAP_SERVICES for `articulate`
so update this.

```bash
cf update-user-provided-service attendee-service-ups -p uri <<< "http://attendee-service-${INITIALS}.apps.internal:8080/attendees"
cf restart articulate
```

Things to note:
- Data entry via `articulate` is still possible, even though `attendee-service` 
is no longer navigable from a browser
- Port 8080 is used _outside_ of the container, even when running at scale 
(software defined networking)
- We no longer require Eureka to help with container-to-container networking
- Try running `cf ssh articulate -c "ps -ef | grep envoy"`
