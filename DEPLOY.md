Opensource enduser response time monitoring setup

**Step 1) Get k8s**

a) BYOK8S (GKE, AKS, EKS, Openshift or any other distribution with persistent storage)
````
kubectl create ns turbonomic
kubectl create -f https://raw.githubusercontent.com/turbonomic/t8c-install/master/operator/deploy/service_account.yaml -n turbonomic
kubectl create -f https://raw.githubusercontent.com/turbonomic/webdriver_exporter/master/deploy/webdriver_yamls/role_binding.yaml -n turbonomic
kubectl create -f https://raw.githubusercontent.com/turbonomic/t8c-install/master/operator/deploy/crds/charts_v1alpha1_xl_crd.yaml -n turbonomic
kubectl create -f https://raw.githubusercontent.com/turbonomic/t8c-install/master/operator/deploy/operator.yaml -n turbonomic
````

b) Download the Turbonomic 7.17 OVA from <http://download.vmturbo.com/appliance/release/7.17.0/turbonomic-t8c-7.17.0-20190708133617000.ova>

Configure the IP address for k8s after booting the VM up by changing this line in /opt/local/etc/turbo.conf:
````
node="10.0.2.15"
````
and change the role that enables Prometheus to run
````
curl -s https://raw.githubusercontent.com/turbonomic/webdriver_exporter/master/deploy/webdriver_yamls/role_binding.yaml > /opt/kubernetes/operator/deploy/role_binding.yaml
````
Bring up kubernetes
````
/opt/local/bin/t8c.sh
````
Please see the install guide pdf for more details at <https://docs.turbonomic.com/pdfdocs/Turbonomic_INSTALL_PRINT_7.17.1.pdf>

**Step 2) Configure prometheus**
````
curl -s https://raw.githubusercontent.com/turbonomic/webdriver_exporter/master/deploy/webdriver_yamls/eum.yaml >  /opt/kubernetes/operator/deploy/crds/eum.yaml
````
Configure the IP address for the ingress it up by changing this line in /opt/kubernetes/operator/deploy/crds/eum.yaml:
````
externalIP: 10.0.2.15
````

and add the customer's web application urls to be monitored by prometheus:
````
          - job_name: 'webdriver'
            metrics_path: /probe
            static_configs:
              - targets:
                - https://10.16.172.11/u/app/index.html
                - https://10.16.172.12/u/app/index.html
````
and apply the configuration
````
kubectl apply -f /opt/turbonomic/kubernetes/operator/deploy/crds/eum.yaml
````

**Step 3) Deploy webdriver from <https://github.com/turbonomic/webdriver_exporter/tree/master/deploy>**
````
cd /opt;  git clone https://github.com/turbonomic/webdriver_exporter.git;
cd webdriver_exporter/deploy; helm install webdriver --name webdriver --namespace turbonomic
````
**Step 4) Deploy prometurbo from <https://github.com/turbonomic/prometurbo/tree/master/deploy>**
````
cd /opt;  git clone https://github.com/turbonomic/prometurbo.git;
cd prometurbo/deploy; helm install prometurbo --name prometurbo --namespace turbonomic --set serverMeta.turboServer=https://10.16.172.10 --set restAPIConfig.opsManagerUserName=administrator --set restAPIConfig.opsManagerPassword=password --set prometurboTargetConfig.createProxyVM=true --set prometurboTargetConfig.targetAddress=http://prometheus-server:9090
````
The Turbo server can either be an already existing classic Turbonomic 6 instance or you can point to a Turbonomic 7 instance that you just created as long as you have a valid admin user/password.