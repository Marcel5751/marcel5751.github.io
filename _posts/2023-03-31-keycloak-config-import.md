---
layout: post
title: "Import KeyCloak Configuration via CI/CD"
---

In a research project, we recently introduced the identity and access management system KeyCloak into our prototype. Since we want to deploy this setup to multiple servers (acceptance and production environments) we thought about how to transfer the KeyCloak Real Settings (Email SMTP configuration, clients, etc.) from one environment to the next. This has 3 main advantages:
 - save the time to enter the same settings again, that were already tested 
 - avoid mistakes when recreating those settings
 - have the base Keycloak realm settings under version control

 <!--more-->

Note: The following information relates to KeyCloak version 21 Quakus distribution, ie. [this version](https://quay.io/repository/keycloak/keycloak?tab=tags&tag=21.0).
On older/newer versions the behavior is (probably) different.

KeyCloak provides several options to export and import Realms. Since I found it quite confusing (partly due to the lackluster documentation) to understand the differences between the mechanisms, I want to share the insights that I gained through research and experimentation:

The KeyCloak GUI has a Realm export option, but this export is incomplete, as it does not contain e.g. the secrets of the realm. Therefore this was not an option.

When running KeyCloak in a docker container, I can recommend the following export command:
```shell
docker exec <CONTAINER_NAME> sh -c '/opt/keycloak/bin/kc.sh export --dir /opt/keycloak/export --realm <REAL_TO_EXPORT>'
docker cp <CONTAINER_NAME>:/opt/keycloak/export/<REAL_TO_EXPORT>-realm.json .
```

Our 3 main requirements were:
 - Substitute variables in generic "template" config: The Clients for each environment are  very similar, they differ mostly in the `redirectUris`
 - Should NOT override existing realm (otherwise on each new deployment, the settings would be "reset" to the realm.json currently stored in git)
 - Users should NOT be part of the exported realm, as they will be different for each environment


Then I had a look at the admin CLI/ REST API (those are pretty much identical in functionality). There are 3 different CLI commands:
 - Passing the `--import-realm` to the `start` command, i.e. `/opt/keycloak/bin/kc.sh start --optimized --import-realm`
 - Using the `import` command, e.g. `/opt/keycloak/bin/kc.sh import --dir /opt/keycloak/data/import --override false`
 - Using the KeyCloak Admin CLI

| Feature/ Keycloak command                                | kc.sh start --import-realm | kc.sh import | kcadm.sh (admin-cli) |
|----------------------------------------------------------|----------------------------|--------------|----------------------|
| Automatic env subst (If No, manual envsubst is required) | yes                        | no           | no                   |
| Override configurable                                    | no (will never override)   | yes          | yes                  |
| Partial Import (import users in a separate step)           | no                         | no           | yes                  |


Therefore, simply using the `kc.sh start --import-realm` option fitted our needs for the initial config setup quite nicely, as it will not override existing realms and it will also handle the environment substitution for us (see [docs](https://www.keycloak.org/server/importExport#_importing_a_realm_during_startup)).

For example, we manually substituted the Callback URIs of a Keycloak client and the SMTP password, as those should be configurable for each environment. Excerpt from the `realm.json` file:
{% highlight json %}
  ...
  {
    "clientId": "some-client",
    ...
    "redirectUris": [
        "${CLIENT_BASE_URL}/callback",
    ],
    ...
  }
  ...

  "smtpServer": {
    ...
    "password": "${SMTP_PASSWORD}",
    ...
  }
{% endhighlight %}

In theory, it is possible to deploy Keycloak without a custom Dockerfile but it recommended to optimize Keycloak performance by handling the build phase beforehand, as stated [in the docs](https://www.keycloak.org/server/configuration#_create_an_optimized_keycloak_build). It is required to explicitly run the Keycloak build stage in order to properly configure the configured Database vendor (KC_DB), in our case MariaDB, See [this discussion](https://groups.google.com/g/keycloak-user/c/wtCpDiFD70U).

Anyway, inside the Dockerfile, we include the json file of the realm to be imported.

{% highlight Dockerfile %}
# This Dockerfile is based on the Keycloak docs: https://www.keycloak.org/server/containers#_building_your_optimized_keycloak_docker_image
FROM quay.io/keycloak/keycloak:21.0.1@sha256:057e1264cae9adbd9be65235d0c837087b0accc183275803b7da81b1b7a2a94c as builder

# Enable health check support. Metrics are also enabled for monitoring of database status
ENV KC_HEALTH_ENABLED=true
ENV KC_METRICS_ENABLED=true

# Configure mariadb database vendor
ENV KC_DB=mariadb

WORKDIR /opt/keycloak
RUN /opt/keycloak/bin/kc.sh build

FROM quay.io/keycloak/keycloak:21.0.1@sha256:057e1264cae9adbd9be65235d0c837087b0accc183275803b7da81b1b7a2a94c
COPY --from=builder /opt/keycloak/ /opt/keycloak/

COPY ./keycloak/data/import/template-realm.json /opt/keycloak/data/import/template-realm.json
{% endhighlight %}

When using the image in the docker-compose.yml, all that is needed is to set the correct entrypoint and pass the environment variables to be replaced:

{% highlight yaml %}
    keycloak:
      environment:
        SMTP_PASSWORD: "<SMTP_PASSWORD>"
      entrypoint:
      ["/opt/keycloak/bin/start.sh", "start", "--optimized", "--import-realm"]
{% endhighlight %}
