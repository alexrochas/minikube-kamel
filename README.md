# Minikube Kamel
> Minor POC with Kamel integrations running on Minikube

Camel-K is the new way to use the power of serverless and Camel.

With Camel-K now is possible to upload to Kubernetes (or Openshift) a route (called in this context "integration") and use it as a lambda function.

## How

First of all, install [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) (and kubectl).

Start minikube (in my case, using virtualbox as driver):
```bash
~$ minikube start --vm-driver=virtualbox
```
> at some point I had problems because I was starting minikube as root user

Once minikube is running start the [kubernetes dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/):
```bash
~$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

Now what we should do to use Camel-K.

Enable minikube registry:
```bash
~$ minikube addons enable registry
registry was successfully enabled
```

Download [Kamel](https://github.com/apache/camel-k/releases) release.

Install it:
```bash
~/path/to/kamel/$ ./kamel install --cluster-setup
```

In the Kubernetes dashboard now we will have new pods and deployments.

## Running

In this repo we have the `routes-rest.js` example that expose a rest-api.

To deploy and attach to the log output:
```bash
~/path/to/project/$ ~/path/to/kamel/kamel run --name=withrest --dependency=camel-undertow routes-rest.js --dev
```

* --dev, attach to logs output and enable hotdeploy
* --name, add a custom name to the integration (filename by default)
* --dependency, explicity an package dependency

> in Java, --dependency is "almost" desnecessary to use this because Kamel recognize and download by it self

The output will be something similar to:
```
[1] 2018-11-10 18:21:04.861 INFO  [main] Runtime - Routes: file:/etc/camel/conf/routes-rest.js
[1] 2018-11-10 18:21:04.865 INFO  [main] Runtime - Language: js
[1] 2018-11-10 18:21:05.813 INFO  [main] DefaultTypeConverter - Type converters loaded (core: 195, classpath: 6)
[1] 2018-11-10 18:21:05.832 INFO  [main] DefaultCamelContext - Apache Camel 2.22.1 (CamelContext: camel-1) is starting
[1] 2018-11-10 18:21:05.834 INFO  [main] ManagedManagementStrategy - JMX is enabled
[1] 2018-11-10 18:21:06.041 INFO  [main] DefaultCamelContext - StreamCaching is not in use. If using streams then its recommended to enable stream caching. See more details at http://camel.apache.org/stream-caching.html
[1] 2018-11-10 18:21:06.153 INFO  [main] DefaultCamelContext - Route: js started and consuming from: timer://js?period=1s
[1] 2018-11-10 18:21:06.168 INFO  [main] DefaultUndertowHost - Starting Undertow server on http://0.0.0.0:8080
[1] 2018-11-10 18:21:06.183 INFO  [main] xnio - XNIO version 3.3.8.Final
[1] 2018-11-10 18:21:06.189 INFO  [main] nio - XNIO NIO Implementation Version 3.3.8.Final
[1] 2018-11-10 18:21:06.264 INFO  [main] DefaultCamelContext - Route: route1 started and consuming from: http://0.0.0.0:8080/say/hello?httpMethodRestrict=GET%2COPTIONS&matchOnUriPrefix=false
[1] 2018-11-10 18:21:06.265 INFO  [main] DefaultCamelContext - Total 2 routes, of which 2 are started
[1] 2018-11-10 18:21:06.266 INFO  [main] DefaultCamelContext - Apache Camel 2.22.1 (CamelContext: camel-1) started in 0.433 seconds
```

Once started with the success we still not able to hit it because we don't have access to the pod where our integration is running.

Exposing port:
```bash
~$ kubectl expose deployment withrest --name withrest-lb --type=LoadBalancer --port 8080
service/withrest-lb exposed
```

Fetch the url for our integration:
```bash
~$ minikube service withrest-lb --url
minikube service withrest-lb --url
```

Or taking a shortcut:
```bash
~$ curl $(minikube service withrest-lb --url)/say/hello
Hello World
```

## Why?

Be able to use the Camel DSL and all the components by itself already look like a really good reason to use it.
Not even talking about the [EIP](http://camel.apache.org/eip.html) but all the uses this could have like an alternative to ETL tools like [Kettle](https://github.com/pentaho/pentaho-kettle) or with something like [Kafka Streams](https://kafka.apache.org/documentation/streams/)
could lead to a very fast design with high decoupled systems.

Truth is, the project is in a very early stage, now is the time to contribute.

## Roadmap

* write integrations using more components and languages

## Meta

Alex Rocha - [about.me](http://about.me/alex.rochas)
