# ecs-docker-run

A simple command line utility to run docker images on Amazon ECS. This utility will generate a task definition based upon the
docker run command line options you are familiar with.

The utility can be used to generate a task definition or to run a task or service in ECS based on this definition.

## options

```
--generate-only - - will only generate the task definition on standard output, without starting anything
--run-as-service - runs the task as service, ECS will ensure that 'desired-count' tasks will keep running
--desired-count - specifies the number tasks to run (default = 1)
--cluster - the ECS cluster to run the task or service (default = cluster)
```

All the following Docker run command line options of Docker 1.7 are supported.

```
-P - publishes all ports by pulling and inspecting the image.
--name - the family name of the task. If unspecified the name will be derived from the image name.
-p - add a port publication to the task definition.
--env - set the environment variable.
--memory - sets the amount of memory to allocate, defaults to 256
--cpu-shares - set the share cpu to allocate, defaults to 100
--entrypoint - changes the entrypoint for the container
--link - set the container link.
-v - set the mount points for the container.
--volumes-from - sets the volumes to mount.
```
all other options are ignored with a warning.


## pre-requisites
the ecs-docker-run utility has the following pre-requisites.

- jq installed
- aws CLI installed version 1.7.44 or higher
- aws connectivity configured (aws iam get-user)
- docker connectivity configured (docker ps)

## usage

### Generating a task definition

```
$ ecs-docker-run --generate-only --name paas-monitor -P  mvanholsteijn/paas-monitor:latest

{
  "family": "paas-monitor",
  "containerDefinitions": [
    {
      "volumesFrom": [],
      "portMappings": [
        {
          "hostPort": 0,
          "containerPort": 1337
        }
      ],
      "command": [],
      "environment": [],
      "links": [],
      "mountPoints": [],
      "essential": true,
      "memory": 256,
      "name": "paas-monitor",
      "cpu": 100,
      "image": "mvanholsteijn/paas-monitor:latest"
    }
  ],
  "volumes": []
}
```


### Starting docker container as a task

```
$ ecs-docker-run \
	--cluster default \
	--name paas-monitor \
	--env SERVICE_NAME=paas-monitor \
	--env SERVICE_TAGS=http \
	--env "MESSAGE=Hello from ECS task" \
	--env RELEASE=v10 \
	-P  \
	mvanholsteijn/paas-monitor:latest
```

### Starting docker container as a service in ECS

```
$ ecs-docker-run \
	--run-as-service \
	--number-of-instances 3 \
	--cluster default \
	--name paas-monitor \
	--env SERVICE_NAME=paas-monitor \
	--env SERVICE_TAGS=http \
	--env "MESSAGE=Hello from ECS paas-monitor service" \
	--env RELEASE=v11 \
	-P  \
	mvanholsteijn/paas-monitor:latest
```

## Caveats
- Multiple container definitions are not supported.
