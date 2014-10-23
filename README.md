# ![](https://raw.github.com/aptible/straptible/master/lib/straptible/rails/templates/public.api/icon-60px.png) Joe Cool

![](https://quay.io/repository/aptible/joecool/status?token=c28d9560-c6f8-43bd-8b8c-7111009c47c0)

Forwards logs from a set of Docker containers to [Gentleman Jerry](https://github.com/aptible/gentlemanjerry).

Joe Cool is implemented as a [logstash-forwarder](https://github.com/elasticsearch/logstash-forwarder) 
instance that tracks a set of docker logs. Docker logs are read by mounting /var/lib/docker/containers 
inside the Joe Cool container.

[Sweetness](https://github.com/aptible/sweetness) spawns Joe Cools to track each set of containers
corresponding to a particular service on a single instance. For example, if one account has 2 web 
processes and 1 worker running on an instance and another account has 1 worker running on the same 
instance, Sweetness would spawn 3 Joe Cools, one for each account-service combination on the instance.

## Example

To see Joe Cool in action, first follow the 
[instructions for building and running Gentleman Jerry](https://github.com/aptible/gentlemanjerry#example) 
so that Joe Cool has somebody to pass the logs to. Following those instructions, you should have a
Gentleman Jerry endpoint and a certificate file. We'll use `gentlemanjerry:1234` for the endpoint and 
`/tmp/jerry-cert/jerry.crt` for the certificate file name in the instructions that follow.

If you don't already have a running container that you'd like to attach logging to, start a bash shell 
in a separate container running using `docker run -i -t quay.io/aptible/ubuntu:14.04`. We'll need a 
prefix of the docker id of this container to monitor it. We use `deadbeef` for this prefix below. After 
we set up Joe Cool and Gentleman Jerry to watch this container, anything you type in the bash shell will
get logged.

Next, pull the image from quay (`docker pull quay.io/aptible/joecool`) or build it locally
(`make build`). The image name will be `quay.io/aptible/joecool:latest` if you pull or build
from the `master` branch.

Finally, run the `run-joe-cool.sh` script in a container created from the resulting image:

```
$ docker run -i -t \
>   -e "LOGSTASH_ENDPOINT=gentlemanjerry:1234" \
>   -e "LOGSTASH_CERTIFICATE=`cat /tmp/jerry-cert/jerry.crt`" \
>   -e "CONTAINERS_TO_MONITOR=deadbeef" \
>   -v /var/lib/docker/containers:/tmp/dockerlogs:ro \
>   quay.io/aptible/joecool:latest
```

You should now see any output in the `deadbeef` container picked up by Joe Cool and sent to Gentleman 
Jerry.

## Environment variables

Runtime behavior of Joe Cool can be modified by passing the following environment variables to 
`docker run`:

* `LOGSTASH_ENDPOINT`: Required. The endpoint where Gentleman Jerry is running.
* `LOGSTASH_CERTIFICATE`: Required. Gentleman Jerry's certificate.
* `CONTAINERS_TO_MONITOR`: Required. A comma-separated list of docker container id prefixes. Specifies 
   which containers Joe Cool will monitor for logs.
* `SERVICE_NAME`: Optional. Correpsonds to the "service" field reported to Gentleman Jerry. Default 
   is '*'.

## Tests

All tests are implemented in bats. Run them with:

    make build

## Copyright

Copyright (c) 2014 [Aptible](https://www.aptible.com). All rights reserved.

[<img src="https://s.gravatar.com/avatar/c386daf18778552e0d2f2442fd82144d?s=60" style="border-radius: 50%;" alt="@aaw" />](https://github.com/aaw)
