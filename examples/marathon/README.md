Wire a local marathon instance into a mesos stack (e.g. mesos-centos);
saves cluttering the stack with frameworks.

# OSX instructions

    brew install mesos (for libmesos.dylib)

get your tarball:

    export M_VER=0.7.5

    curl -s http://downloads.mesosphere.io/marathon/v${M_VER}/marathon-${M_VER}.tgz | tar zxv

    export LIBPROCESS_IP=10.0.0.1 ## mac IP on the host network
    export ZK=zk://master1:2181
    ./marathon-${M_VER}/bin/start --zk ${ZK}/marathon --master ${ZK}/mesos --event_subscriber http_callback \
    --ha --checkpoint --task_launch_timeout 300000

NB: the --task_launch_timeout (in milliseconds) should match the slaves 
--executor_registration_timeout option.

ui is at:

     open http://localhost:8080/

# HA testing

All you'd have to do is run marathon out of the same directory N times and pass a new --http_port option i.e.

    for port in 8080 8081 8082
      do 
        ./marathon-${M_VER}/bin/start --zk ${ZK}/marathon --master ${ZK}/mesos --ha --event_subscriber http_callback --http_port $port --task_launch_timeout 300 &
      done

After they spin up, you can treat any of them as a master - the other nodes just proxy to the master marathon.

# deploying some apps
basic date loop in a busybox image:

    curl -X POST -H "Content-Type: application/json" http://localhost:8080/v2/apps -d@bbdate.json

try a docker image with networking:

    curl -X POST -H "Content-Type: application/json" http://localhost:8080/v2/apps -d@redis.json

check we can run 'vanilla' executors (i.e. not dockerized)

    curl -X POST -H "Content-Type: application/json" http://localhost:8080/v2/apps -d@pythons.json



## Docker notes:

first off, "uri" is empty. docker will pull the image for us
and we don't need anything else.

These settings tell Docker to map the container redis port
to a random host port on the slave:

      "network": "BRIDGE",
      "portMappings": [
        { "containerPort": 6379, "hostPort": 0, "protocol": "tcp"}
      ]


then we map that host port to a global app port with the usual:

      port: [0] 

## scaling

curl -XPUT http://localhost:8080/v2/apps/redis -d '{ "instances": "3" }' -H "Content-type: application/json"

# healthchecks

This sample app responds to the configured healthcheck with a 200. A 'GET /toggle' will flip it between
passing and failing). Click the app to get little traffic lights showing health or otherwise.

    curl -X POST -H "Content-Type: application/json" http://localhost:8080/v2/apps -d@http-health.json

