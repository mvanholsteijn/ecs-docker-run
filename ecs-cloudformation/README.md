# ECS Cloudformation

This directory contains a ECS cloudformation template, which is a slightly modified version of the
default ECS cluster that is created by the ECS first-run wizard.

It creates a cluster with three machines and has a load balancer attached.

to create the Cluster from the run time type the following commands:

``` bash
aws ec2 import-key-pair \
	  --key-name ecs-$USER-key \
          --public-key-material  "$(ssh-keygen -y -f ~/.ssh/id_rsa)" 

aws cloudformation create-stack \
	--stack-name ecs-$USER-cluster \
	--template-body "$(<ecs.json)"  \
	--capabilities CAPABILITY_IAM \
	--parameters \
		ParameterKey=KeyName,ParameterValue=ecs-$USER-key \
		ParameterKey=EcsClusterName,ParameterValue=ecs-$USER-cluster 

function waitOnCompletion() {
	STATUS=IN_PROGRESS
	while expr "$STATUS" : '^.*PROGRESS' > /dev/null ; do
		sleep 10
		STATUS=$(aws cloudformation describe-stacks --stack-name ecs-$USER-cluster | jq -r '.Stacks[0].StackStatus')
		echo $STATUS
	done
}

waitOnCompletion

aws ecs create-cluster --cluster-name ecs-$USER-cluster
```

Now the cluster is ready, you can start the infamous paas-monitor application!

``` bash
../bin/ecs-docker-run --run-as-service \
			--number-of-instances 3 \
			--cluster ecs-$USER-cluster \
			-p :80:1337 \
			mvanholsteijn/paas-monitor


ELBNAME=$(aws cloudformation describe-stacks --stack-name ecs-$USER-cluster | \
		jq -r '.Stacks[0].Outputs[] | select(.OutputKey =="EcsElbName") | .OutputValue') 

DNSNAME=$(aws elb describe-load-balancers --load-balancer-names $ELBNAME | \
		jq -r .LoadBalancerDescriptions[].DNSName)

#
# Once the paas-monitor is running, you can actually access the application
#
open http://$DNSNAME/status

```

