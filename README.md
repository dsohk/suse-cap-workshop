# SUSE Cloud Application Platform Workshop

## Reference
* https://www.suse.com/documentation/cloud-application-platform-1/index.html
* https://docs.cloudfoundry.org/
* http://cli.cloudfoundry.org/en-US/cf/

## Exercise 1 - Connect to SUSE CAP with CLI or WebUI

### Task 1 – Install Cloud Foundry CLI

SUSE Cloud Application Platform can be connected by the open source Cloud Foundry CLI tool. This tool can be installed on Windows, Mac or Linux. Please follow the instruction on the link below to this cli tool into your PC/laptop.

https://docs.cloudfoundry.org/cf-cli/install-go-cli.html

After the CLI is installed, you should be able to check its version (`cf version`) and get help usage (`cf help -a`).

```
$ cf version
```

### Task 2 - Login with CLI

Refer to the Lab Environment, find the API endpoint and issue the cf login command to login to SUSE Cloud Foundry. Please follow instructor’s instruction to get the assigned user id to login.

Run cf api command to set SUSE Cloud Foundry API End Point first.

```
cf api --skip-ssl-validation https://api.example.com
cf login
```

### Task 3 - Login via WebUI

1. Navigate to the SUSE CAP Web UI URL defined in the lab environment with your browser in your desktop.

2. Skip the warning message about self-signed SSL certification if any.

3. You should see this login page for SUSE Cloud Application Platform such as below.

4. Login with your assigned user credential.

## Exercise 2 - Push directly from WebUI

Try with this url: https://github.com/dsohk/clumsy-bird

## Exercise 3 - Push from CLI

Take a sample hello-world springboot application:

```
git clone https://github.com/dsohk/cf-hello-springboot
cd cf-hello-springboot
cf push
```

Examine `manifest.yml` file

## SUSE Cloud Application Architecture Review

Before we move on, let's review the SUSE CAP architecture.

Inspect the SUSE CAP on kubernetes
```
kubectl get nodes
kubectl get pod –all-namespaces
kubectl get services --all-namespaces |grep Load
```

## Exercise 4 - scaling application

finish exercise 2 if not done yet and continue here.

Scale the application to run in 3 instances
```
cf scale clumpsy-bird -i 3
```

Increase the memory of an application instance
```
cf scale clumpsy-bird -m 156M
```

## Exercise 5 - view application logs

```
cf logs clumpsy-bird --recent
```

## Exercise 6 - ssh into application container instance

```
cf ssh clumpsy-bird
```

Perform linux operations such as `ls -l`, `ps -ef`

## Exercise 7 - Stateful Apps

This exercise will lead you to deploy a stateful app using redis as backend
storage database.

```
git clone https://github.com/scf-samples/cf-redis-example-app
cd cf-redis-example-app
cf push redis-example-app --no-start
```

View marketplace
```
cf marketplace
cf create-service redis 4-0-10 my-redis
cf services
cf bind-service redis-example-app my-redis
cf env redis-example-app
cf start redis-example-app
```

Test the app

Put some data into redis key-value store
```
export APP=redis-example-app.open-cloud.net
curl -X PUT $APP/key1 -d 'data=value1'
curl -X PUT $APP/key2 -d 'data=value2'

curl -X GET $APP/key1
curl -X GET $APP/key2
```

Clean up the app
```
cf stop redis-example-app
cf unbind-service redis-example-app my-redis
cf delete redis-example-app -r
cf delete-service my-redis
```

## Exercise 8 - Blue/Green Deployment

Start from fresh (remove hello-ruby app if exists)
```
cf apps
cf delete hello-ruby-v1 -r
cf delete hello-ruby-v2 -r
```

Download the latest hello-ruby app and deploy via CLI

```
git clone https://github.com/dsohk/cf-hello-ruby
cd cf-hello-ruby
cf push hello-ruby-v1 -n hello-world
curl http://hello-world.open-cloud.net
```

Modify the hello.rb (add the word "again" after hello world message) and save
the file

Push the latest change of the app into a new app name

```
cf push hello-ruby-v2 -n hello-world-v2
```

Direct the traffic from origin routes to the app hello-ruby-v2. At this stage,
traffic will be directed to both old and new apps in round robin fashion.

```
cf map-route hello-ruby-v2 open-cloud.net -n hello-world
```

Remove the old application when it is decided to phase it out.

```
cf unmap-route hello-ruby-v1 open-cloud.net -n hello-world
```

Lastly, remove the old application if it's not used anymore

```
cf delete hello-ruby-v1 -r
```

Clean up the unused routes

```
cf delete-route open-cloud.net --hostname hello-world-v2
```


