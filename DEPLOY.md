Opensource enduser response time monitoring setup

**Step 1) deploy Turbonomic OVA**

a) Download the Turbonomic 7.17.2 OVA from:

<http://download.vmturbo.com/appliance/release/7.17.2/turbonomic-t8c-7.17.2-20190824014505000.ova>

b) Configure the IP address of the VM after booting it up
````
vi /opt/local/etc/turbo.conf
````
And change this line:
````
node="10.16.172.10"
````
and change the role that enables Prometheus to run
````
curl -s <https://raw.githubusercontent.com/esara/webdriver_exporter/master/deploy/webdriver_yamls/role_binding.yaml> > /opt/kubernetes/operator/deploy/role_binding.yaml
````
c) Bring up kubernetes
````
/opt/local/bin/t8c.sh
````
Please see the install guide pdf for more details at <https://docs.turbonomic.com/pdfdocs/Turbonomic_INSTALL_PRINT_7.17.1.pdf>

**Step 2) configure prometheus**
````
curl -s <https://raw.githubusercontent.com/esara/webdriver_exporter/master/deploy/webdriver_yamls/eum.yaml> >  /opt/kubernetes/operator/deploy/crds/eum.yaml
````
and add the customer's web application urls to be monitored by prometheus:
````
          - job_name: 'webdriver'*
            metrics_path: /probe*
            static_configs:*
              - targets:*
                - <https://10.16.172.10/u/app/index.html>*
                - <https://10.16.172.11/u/app/index.html>*
````
and apply the configuration
````
kubectl apply -f /opt/turbonomic/kubernetes/operator/deploy/crds/eum.yaml
````

**Step 3) deploy webdriver from <https://github.com/esara/webdriver_exporter/tree/master/deploy>**
````
cd /opt;  git clone <https://github.com/esara/webdriver_exporter.git>;
cd webdriver_exporter/deploy; helm install webdriver --name webdriver --namespace turbonomic
````
**Step 4) deploy prometurbo from <https://github.com/turbonomic/prometurbo/tree/master/deploy>**
````
cd /opt;  git clone <https://github.com/turbonomic/prometurbo.git>;
cd prometurbo/deploy; helm install prometurbo --name prometurbo --namespace turbonomic --set serverMeta.turboServer=https://10.16.172.10 --set restAPIConfig.opsManagerUserName=administrator --set restAPIConfig.opsManagerPassword=password --set prometurboTargetConfig.createProxyVM=true --set prometurboTargetConfig.targetAddress=[http://prometheus-server:9090](http://prometheus-server:9090/)
````
The Turbo server can either be an already existing classic Turbonomic 6 instance or you can point to a Turbonomic 7 instance that you just created as long as you have a valid admin user/password.