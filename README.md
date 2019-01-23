# fluentd-example

This is an example for fluentd with nginx log. 

Related post at [Medium](https://medium.com/life-of-small-data-engineer/data-pipeline-from-scratch-part1-data-collector-afcd362e0568)
(in chinese).

Image at docke hub [lukehong/nginx-fluentd-example](https://cloud.docker.com/u/lukehong/repository/docker/lukehong/nginx-fluentd-example)

## Basic example

Tail nginx access log then standard output.

Using image with tag `basic` [(basic/Dockerfile)](https://github.com/LukeHong/fluentd-example/blob/master/basic/Dockerfile)

```
git clone git@github.com:LukeHong/fluentd-example.git
cd fluentd-example
mkdir nginx

# start nginx container
docker run -v $(PWD)/nginx:/var/log/nginx/ -d -p 8080:80 --name nginx-server nginx

# start fluentd container
docker run -v $(PWD)/nginx:/fluentd/nginx --rm --name fluentd lukehong/nginx-fluentd-example:basic
```

Use curl to request nginx server for several time, then you will see some log from fluentd.

```
curl http://localhost:8080
```

## Advance example

This example with tail nginx log and forward to another fluentd by `fluentd-forwarder`.
Receive and save to Google Cloud Storage by `fluentd-aggregator`

At first, you need to prepare a Google Cloud Platform(GCP) service account 
with enough permission to access Google Cloud Storage(GCS) bucket,
and copy the service account key file to `advance/aggregator`.

Then change the `[YOUR_PROJECT]` `[YOUR_KEYFILE_NAME]` `[YOUR_BUCKET_NAME]` 
in `advance/aggregator/Dockerfile` and `advance/aggregator/fluent.conf`
to the right value.

After that you can build your own image with `docker build -t fluentd-aggregator advance/aggregator/` command.

Using image with tag `adv-forwarder` [(basic/Dockerfile)](https://github.com/LukeHong/fluentd-example/blob/master/advance/forwarder/Dockerfile)
and the image `fluentd-aggregator` that build by yourself.

```
# start nginx container
docker run -v $(PWD)/nginx:/var/log/nginx/ -d -p 8080:80 --name nginx-server nginx

# start fluentd-aggregator
docker run --name fluentd-aggregator --rm fluentd-aggregator

# start fluentd-forwarder  and link to fluentd-aggregator
docker run --link fluentd-aggregator -v $(PWD)/nginx:/fluentd/nginx -d --rm --name fluentd-forwarder lukehong/nginx-fluentd-example:adv-forwarder
```

```
# request for access log (save to GCS)
curl http://localhost:8080

# request for error log (stdout)
curl http://localhost:8080/error
```
