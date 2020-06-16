# knative-tomcat
Example to setup a "knative" tomcat in OpenShift.

# Setup your OpenShift cluster or use a prepared one (here I have used AWS).
Control the $HOME/.aws/credentials make sure you have
```
[default]
aws_access_key_id = Access Key
aws_secret_access_key = Secret Key
```
Run the install I downloaded it from https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/ and install as following:
```
./openshift-install create cluster --dir=jean-frederic/
export KUBECONFIG=/home/jfclere/TMP/jean-frederic/auth/kubeconfig
```

To destroy the cluster (to prevent paying for example)
```
./openshift-install destroy cluster --dir=jean-frederic/
```
# Install the "OpenShift Servless Operator"
Use the OpenShift Container Platform web console to do that!

See https://docs.openshift.com/container-platform/4.4/serverless/installing_serverless/installing-openshift-serverless.html#serverless-install-web-console_installing-openshift-serverless

# Install Knative Serving.
```
oc create namespace knative-serving
oc apply -f serving.yaml
```

# Check the knative is installed
```
oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'
DependenciesInstalled=True
DeploymentsAvailable=True
InstallSucceeded=True
Ready=True
```
Note that it might take some minutes to have everything ready.

# Create the tomcat serverless application
```
oc new-project tomcat
oc apply -f natservice.yaml

# Check the result:
```
oc get ksvc tomcat
NAME      URL                                                  LATESTCREATED   LATESTREADY    READY     REASON
tomcat    http://tomcat-tomcat.apps.jclere.rhmw-runtimes.net   tomcat-s4zdv    tomcat-s4zdv   True      
curl -v http://tomcat-tomcat.apps.jclere.rhmw-runtimes.net
*   Trying 52.208.140.210:80...
* Connected to tomcat-tomcat.apps.jclere.rhmw-runtimes.net (52.208.140.210) port 80 (#0)
> GET / HTTP/1.1
> Host: tomcat-tomcat.apps.jclere.rhmw-runtimes.net
> User-Agent: curl/7.69.1
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< content-length: 163
< content-type: application/json;charset=UTF-8
< date: Mon, 15 Jun 2020 11:40:24 GMT
< set-cookie: JSESSIONID=34BC7BDA31872E3EDBDA762D982ADA6C; Path=/; HttpOnly
< x-envoy-upstream-service-time: 7838
< server: envoy
< set-cookie: ac3f758b54529f5dc28461a7cf2a1ab7=e8fe7035d84cab843d436962be697a37; path=/; HttpOnly
< cache-control: private
< connection: close
< 
{
  "counter": 1,
  "id": "34BC7BDA31872E3EDBDA762D982ADA6C",
  "new": true,
  "server": "10.129.2.29",
  "hostname": "tomcat-s4zdv-deployment-84d9dc9746-9c9zg"
}
* Closing connection 0
```

# curl and check the counter (replace 34BC7BDA31872E3EDBDA762D982ADA6 by the set-cookie: JSESSIONID=value).
```
curl -v --cookie JSESSIONID=34BC7BDA31872E3EDBDA762D982ADA6C http://tomcat-tomcat.apps.jclere.rhmw-runtimes.net
```
