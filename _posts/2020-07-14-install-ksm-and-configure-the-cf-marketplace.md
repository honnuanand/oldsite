---
layout: post
title: Install Container Services Manager(KSM) and configure Tanzu Application Service for K8s - marketplace
date: 2020-07-14 18:45
category: 
    - Tanzu
author: Anand Rao
tags: 
    - Tanzu
    - k8s
    - services
    - rabbitmq
summary: The Application journey is intwined with the data and services it uses. When it comes to running Apps seamlessly on k8s, Tanzu application service leads the way and KSM comes in as a companion to TAS as a way to bring the services needed by the apps running on TAS. 
---

#### Follow the instructions from  [here](http://docs-pcf-staging.cfapps.io/ksm/0-n/installing-helm.html)

#### 1. Prepare S3 bucket for ksm. Use [my terraform config](https://github.com/honnuanand/s3-terraform) for this if you want. 
Collect the `bucket_id` , `region` and `bucket_regional_domain_name` fields from the terraform output. We will use these fields in the configs of the override.yml later. 
{% highlight bash  %}
bucket_id = rao-ksm-chartmuseum
bucket_regional_domain_name = rao-ksm-chartmuseum.s3.us-west-2.amazonaws.com
region = us-west-2
{% endhighlight %}

#### 2. Download the `ksm` , `ksm-[VERSION].tgz` from pivnet
#### 3. Copy the downloaded files to the working folder. Move the `ksm` cli to the `/usr/local/bin` folder and unquarantine it. 
{% highlight bash  %}
$ cp ~/Downloads/ksm-0.10.30.tgz .
$ sudo cp ~/Downloads/ksm /usr/local/bin
$ sudo chmod +x /usr/local/bin/ksm
$ sudo xattr -r -d com.apple.quarantine /usr/local/bin/ksm
{% endhighlight %}

#### 4. Verify the dev registry access 
{% highlight bash %}
$ docker login dev.registry.pivotal.io -u arao@pivotal.io
Password:

$ docker pull dev.registry.pivotal.io/$ container-services-manager/broker:0.10.62
$ docker pull dev.registry.pivotal.io/$ container-services-manager/daemon:0.10.62
$ docker pull dev.registry.pivotal.io/$ container-services-manager/chartmuseum:v0.12.0
{% endhighlight %}


#### 5. Extract the helm tgz file 
```bash
$ tar -zxvf ksm-0.10.62.tgz
```

#### 6. Prepare a [overrides.yml](https://gist.github.com/honnuanand/0eff87527be19d692eb9f121ef4c96a7) file using the overrides.yaml.template make sure to change the parameters. My install here uses S3 for storage. 
Be aware that the S3 bucket URL you got from earlier should be split into bucket name and endpoint in the overrides.  

#### 7. Install the helm chart 
```bash
$ helm install ksm . --set createPVC=false -f overrides.yaml -n ksm
NAME: ksm
LAST DEPLOYED: Tue Jul 14 12:32:46 2020
NAMESPACE: ksm
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
To configure the KSM CLI:
  export KSM_TARGET=http://daemon.svcs.aws.clue2solve.com:80
  export KSM_USER=admin
  export KSM_PASSWORD=$(kubectl get secret -n ksm ksm-ksm-daemon  -o=jsonpath='{@.data.SECURITY_USER_PASSWORD}' | base64 --decode)

  After configuring the above environment variables, the KSM CLI will report client and server versions:
    ksm version

    For more information about KSM CLI commands see the documentation: https://docs.pivotal.io/ksm/using.html

    For help with some common KSM CLI commands run:
        ksm cluster register --help
        ksm cluster set-default --help
        ksm offer save --help
        ksm offer list --help
```

#### 8. Copy the 3 export lines in the helm install output and execute it 
```bash
export KSM_TARGET=http://daemon.svcs.aws.clue2solve.com:80
export KSM_USER=admin
export KSM_PASSWORD=$(kubectl get secret -n ksm ksm-ksm-daemon  -o=jsonpath='{@.data.SECURITY_USER_PASSWORD}' | base64 --decode)
```




#### 9. Setup the variables to start using ksm 

Create the service account for ksm on the target cluster to be used for the service instances. Create the yaml file and provision it. 
```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: ksm-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: ksm-cluster-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: ksm-admin
    namespace: kube-system
```

```bash
$ kapp deploy -a ksm -f target-cluster-service-account.yml
```

Create some Environment Variables 
( The First line needs to have the cluster name assigned to the variable)

```bash
$ cluster=shared-svc #`grep current $kube_config|sed "s/ //g"|cut -d ":" -f 2`

echo "Using cluster $cluster"

server=`grep -B 2 "name: $cluster" $kube_config \
  |grep server|sed "s/ //g"|sed "s/^[^:]*://g"`

certificate=`grep -B 2 "name: $cluster" $kube_config \
  |grep certificate|sed "s/ //g"|sed "s/.*://"`
Using cluster shared-svcs-admin@shared-svcs

```


Extract the service account into into env variables 
```bash

secret_name=$(kubectl get serviceaccount ksm-admin \
 --namespace=kube-system -o jsonpath='{.secrets[0].name}')

secret_val=$(kubectl --namespace=kube-system get secret $secret_name \
 -o jsonpath='{.data.token}')

secret_val=$(echo ${secret_val} | base64 --decode)
```


Create a target cluster credential file. 
```bash
cat > cluster-creds.yaml << EOF
token: ${secret_val}
server: ${server}
caData: ${certificate}
EOF
```

Now register the target cluster on ksm. 
```bash
$ ksm cluster register ksm-svcs ./cluster-creds.yaml
```
#### 10. This is the step where you will need to prepare the offer.  ( I will skip this for now and use a pre-prepared offer.) I will come back and add comments around how to create an offer. 

#### 11. Execute the offer save command in the folder which has the helm chart and the path to the offer/plan files. 
```bash
anandrao at Anands-MBP-3 in ~/pivotal/repos/ksm-sample/ci-charts (master)
$ ksm offer save ./ksm-rabbit rabbitmq-6.18.1.tgz
```
Once the offer is saved ,  you can run the list command to check the offers available.
```bash
$ ksm offer list
MARKETPLACE NAME  INCLUDED CHARTS VERSION PLANS
rabbitmq                rabbitmq        6.18.1  [persistent ephemeral]
mysql                   mysql           1.3.0   [medium small]
```

#### 12. Now,  switch to your CF  side and create  the broker 
(if you have configured the CF related section on step as in the overrides.yml, KSM will automatically do the broker registration for you. KSM will also push the new updates to the offers as well.)
```bash
$ cf create-service-broker ksm-uno admin SNIP https://broker.svcs.aws.clue2solve.com
$ cf service-brokers
Getting service brokers as admin...

name      url
ksm-uno   https://broker.svcs.aws.clue2solve.com
```
#### 13. List and enable the service access 
```bash
$ cf service-access
Getting service access as admin...
broker: ksm-uno
   service    plan         access   orgs
   mysql      medium       all
   mysql      small        all
   rabbitmq   ephemeral    all
   rabbitmq   persistent   all

$ cf enable-service-access rabbitmq
Enabling access to all plans of service rabbitmq for all orgs as admin...
OK

```

#### 14. Time to create the service instances 
```bash
anandrao at Anands-MBP-3 in ~/pivotal/repos/tas4k8s-playground/tkg (master●)
$ cf services
Getting services in org test / space test as admin...

No services found

$ cf create-service rabbitmq ephemeral my-rabbit -c '{"service":{"type":"LoadBalancer"}}'
Creating service instance my-rabbit in org test / space test as admin...
OK

Create in progress. Use 'cf services' or 'cf service my-rabbit' to check operation status.

$ cf create-service mysql small mysql-uno
Creating service instance mysql-uno in org test / space test as admin...
OK

Create in progress. Use 'cf services' or 'cf service mysql-uno' to check operation status.

$ cf services
Getting services in org test / space test as admin...

name        service    plan        bound apps   last operation       broker    upgrade available
my-rabbit   rabbitmq   ephemeral                create in progress   ksm-uno   no
mysql-uno   mysql      small                    create in progress   ksm-uno   no

$ cf services
Getting services in org test / space test as admin...

name        service    plan        bound apps   last operation       broker    upgrade available
my-rabbit   rabbitmq   ephemeral                create succeeded     ksm-uno   no
mysql-uno   mysql      small                    create in progress   ksm-uno   no
```

After this ,  you can now  use the common CF constructs to bind the service to the App and use it.  Enjoying Developing and deploying on TAS with KSM. 