---

copyright:
  years: 2014, 2017
lastupdated: "2017-08-13"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}


# Deploying apps in clusters
{: #cs_apps}

You can use Kubernetes techniques to deploy apps and to ensure your apps are up and running at all times. For example, you can perform rolling updates and rollbacks without downtime for your users.
{:shortdesc}

Deploying an app generally includes the following steps.

1.  [Install the CLIs](cs_cli_install.html#cs_cli_install).

2.  Create a configuration script for your app. [Review the best practices from Kubernetes. ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/concepts/configuration/overview/)

3.  Run the configuration script by using one of the following methods.
    -   [The Kubernetes CLI](#cs_apps_cli)
    -   The Kubernetes dashboard
        1.  [Start the Kubernetes dashboard.](#cs_cli_dashboard)
        2.  [Run the configuration script.](#cs_apps_ui)


## Launching the Kubernetes dashboard
{: #cs_cli_dashboard}

Open a Kubernetes dashboard on your local system to view information about a cluster and its worker nodes.
{:shortdesc}

Before you begin, [target your CLI](cs_cli_install.html#cs_cli_configure) to your cluster. This task requires the [Administrator access policy](cs_cluster.html#access_ov). Verify your current [access policy](cs_cluster.html#view_access).

You can use the default port or set your own port to launch the Kubernetes dashboard for a cluster.
-   Launch your Kubernetes dashboard with the default port 8001.
    1.  Set the proxy with the default port number.

        ```
        kubectl proxy
        ```
        {: pre}

        Your CLI output looks as follows:

        ```
        Starting to serve on 127.0.0.1:8001
        ```
        {: screen}

    2.  Open the following URL in a web browser to see the Kubernetes dashboard.

        ```
        http://localhost:8001/ui
        ```
        {: codeblock}

-   Launch your Kubernetes dashboard with your own port.
    1.  Set the proxy with your own port number.

        ```
        kubectl proxy -p <port>
        ```
        {: pre}

    2.  Open the following URL in a browser.

        ```
        http://localhost:<port>/ui
        ```
        {: codeblock}


When you are done with the Kubernetes dashboard, use `CTRL+C` to exit the `proxy` command.

## Allowing public access to apps
{: #cs_apps_public}

To make an app publicly available, you must update your configuration script before you deploy the app into a cluster.
{:shortdesc}

Depending on whether you created a lite or a standard cluster, different ways exist to make your app accessible from the internet.

<dl>
<dt><a href="#cs_apps_public_nodeport" target="_blank">Service of type NodePort</a> (lite and standard clusters)</dt>
<dd>Expose a public port on every worker node and use the public IP address of any worker node to publicly access your service in the cluster. The public IP address of the worker node is not permanent. When a worker node is removed or re-created, a new public IP address is assigned to the worker node. You can use the service of type NodePort for testing the public access for your app or when public access is needed for a short amount of time only. When you require a stable public IP address and more availability for your service endpoint, expose your app by using a service of type LoadBalancer or Ingress.</dd>
<dt><a href="#cs_apps_public_load_balancer" target="_blank">Service of type LoadBalancer</a> (standard clusters only)</dt>
<dd>Every standard cluster is provisioned with 4 portable public IP addresses that you can use to create an external TCP/ UDP load balancer for your app. You can customize your load balancer by exposing any port that your app requires. The portable public IP address that is assigned to the load balancer is permanent and does not change when a worker node is re-created in the cluster.

</br>
If you need HTTP or HTTPS load balancing for your app and want to use one public route to expose multiple apps in your cluster as services, use the built-in Ingress support with {{site.data.keyword.containershort_notm}}.</dd>
<dt><a href="#cs_apps_public_ingress" target="_blank">Ingress</a> (standard clusters only)</dt>
<dd>Expose multiple apps in your cluster by creating one external HTTP or HTTPS load balancer that uses a secured and unique public entrypoint to route incoming requests to your apps. Ingress consists of two main components, the Ingress resource and the Ingress controller. The Ingress resource defines the rules for how to route and load balance incoming requests for an app. All Ingress resources must be registered with the Ingress controller that listens for incoming HTTP or HTTPS service requests and forwards requests based on the rules defined for each Ingress resource. Use Ingress if you want to implement your own load balancer with custom routing rules and if you need SSL termination for your apps.

</dd></dl>

### Configuring public access to an app by using the NodePort service type
{: #cs_apps_public_nodeport}

Make your app publicly available by using the public IP address of any worker node in a cluster and exposing a node port. Use this option for testing and short-term public access.
{:shortdesc}

You can expose your app as a Kubernetes service of type NodePort for lite or standard clusters.

For {{site.data.keyword.Bluemix_notm}} Dedicated environments, public IP addresses are blocked by a firewall. To make an app publicly available, use a [service of type LoadBalancer](#cs_apps_public_load_balancer) or [Ingress](#cs_apps_public_ingress) instead.

**Note:** The public IP address of a worker node is not permanent. If the worker node must be re-created, a new public IP address is assigned to the worker node. If you need a stable public IP address and more availability for your service, expose your app by using a [service of type LoadBalancer](#cs_apps_public_load_balancer) or [Ingress](#cs_apps_public_ingress).




1.  Define a [service ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/concepts/services-networking/service/) section in the configuration script.
2.  In the `spec` section for the service, add the NodePort type.

    ```
    spec:
      type: NodePort
    ```
    {: codeblock}

3.  Optional: In the `ports` section, add a NodePort in the 30000 - 32767 range. Do not specify a NodePort that is already in use by another service. If you are unsure which NodePorts are already in use, do not assign one. If no NodePort is assigned, a random one is assigned for you.

    ```
    ports:
      - port: 80
        nodePort: 31514
    ```
    {: codeblock}

    If you want to specify a NodePort and want to see which NodePorts are already in use, you can run the following command.

    ```
    kubectl get svc
    ```
    {: pre}

    Output:

    ```
    NAME           CLUSTER-IP     EXTERNAL-IP   PORTS          AGE
    myapp          10.10.10.83    <nodes>       80:31513/TCP   28s
    redis-master   10.10.10.160   <none>        6379/TCP       28s
    redis-slave    10.10.10.194   <none>        6379/TCP       28s
    ```
    {: screen}

4.  Save the changes.
5.  Repeat to create a service for every app.

    Example:

    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: my-nodeport-service
      labels:
        run: my-demo
    spec:
      selector:
        run: my-demo
      type: NodePort
      ports:
       - protocol: TCP
         port: 8081
         # nodePort: 31514

    ```
    {: codeblock}

**What's next:**

When the app is deployed, you can use the public IP address of any worker node and the NodePort to form the public URL to access the app in a browser.

1.  Get the public IP address for a worker node in the cluster.

    ```
    bx cs workers <cluster_name>
    ```
    {: pre}

    Output:

    ```
    ID                                                Public IP   Private IP    Size     State    Status
    prod-dal10-pa215dcf5bbc0844a990fa6b0fcdbff286-w1  192.0.2.23  10.100.10.10  u1c.2x4  normal   Ready
    prod-dal10-pa215dcf5bbc0844a990fa6b0fcdbff286-w2  192.0.2.27  10.100.10.15  u1c.2x4  normal   Ready
    ```
    {: screen}

2.  If a random NodePort was assigned, find out which one was assigned.

    ```
    kubectl describe service <service_name>
    ```
    {: pre}

    Output:

    ```
    Name:                   <service_name>
    Namespace:              default
    Labels:                 run=<deployment_name>
    Selector:               run=<deployment_name>
    Type:                   NodePort
    IP:                     10.10.10.8
    Port:                   <unset> 8080/TCP
    NodePort:               <unset> 30872/TCP
    Endpoints:              172.30.171.87:8080
    Session Affinity:       None
    No events.
    ```
    {: screen}

    In this example, the NodePort is `30872`.

3.  Form the URL with one of the worker node public IP addresses and the NodePort. Example: `http://192.0.2.23:30872`

### Configuring public access to an app by using the load balancer service type
{: #cs_apps_public_load_balancer}

Expose a port and use a portable public IP address for the load balancer to access the app. Unlike with NodePort service, the portable public IP address of the load balancer service is not dependent on the worker node that the app is deployed on. The portable public IP address of the load balancer is assigned for you and does not change when you add or remove worker nodes, which means that load balancer services are more highly available than NodePort services. Users can select any port for the load balancer and are not limited to the NodePort port range. You can use load balancer services for TCP and UDP protocols.

When a {{site.data.keyword.Bluemix_notm}} Dedicated account is [enabled for clusters](cs_ov.html#setup_dedicated), you can request public subnets to be used for load balancer IP addresses. [Open a support ticket](/docs/support/index.html#contacting-support) to create the subnet, and then use the [`bx cs cluster-subnet-add`](cs_cli_reference.html#cs_cluster_subnet_add) command to add the subnet to the cluster.

**Note:** Load balancer services do not support TLS termination. If your app requires TLS termination, you can expose your app via [Ingress](#cs_apps_public_ingress), or configure your app to manage the TLS termination.

Before you begin:

-   This feature is available for standard clusters only.
-   You must have a portable public IP address available to assign to the load balancer service.

To create a load balancer service:

1.  [Deploy your app to the cluster](#cs_apps_cli). When you deploy your app to the cluster, one or more pods are created for you that run your app in a container. Ensure that you add a label to your deployment in the metadata section of your configuration script. This label is needed to identify all pods where your app is running, so that they can be included in the load balancing.
2.  Create a load balancer service for the app that you want to expose. To make your app available on the public internet, you must create a Kubernetes service for your app and configure your service to include all the pods that make up your app into the load balancing.
    1.  Open your preferred editor and create a service configuration script that is named, for example, `myloadbalancer.yaml`.
    2.  Define a load balancer service for the app that you want to expose to the public.

        ```
        apiVersion: v1
        kind: Service
        metadata:
          name: <myservice>
        spec:
          type: LoadBalancer
          selector:
            <selectorkey>:<selectorvalue>
          ports:
           - protocol: TCP
             port: 8080
        ```
        {: codeblock}

        <table>
        <thead>
        <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
        </thead>
        <tbody>
        <tr>
        <td><code>name</code></td>
        <td>Replace <em>&lt;myservice&gt;</em> with a name for your load balancer service.</td>
        </tr>
        <tr>
        <td><code>selector</code></td>
        <td>Enter the label key (<em>&lt;selectorkey&gt;</em>) and value (<em>&lt;selectorvalue&gt;</em>) pair that you want to use to target the pods where your app runs. For example, if you use the following selector <code>app: code</code>, all pods that have this label in their metadata are included in the load balancing. Enter the same label that you used when you deployed your app to the cluster. </td>
         </tr>
         <td><code>port</code></td>
         <td>The port that the service listens on.</td>
         </tbody></table>
    3.  Optional: If you want to use a specific portable public IP address for your load balancer that is available to your cluster, you can specify that IP address by including the `loadBalancerIP` in the spec section. For more information, see the [Kubernetes documentation ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/concepts/services-networking/service/).
    4.  Optional: You might choose to configure a firewall by specifying the `loadBalancerSourceRanges` in the spec section. For more information, see the [Kubernetes documentation ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/tasks/access-application-cluster/configure-cloud-provider-firewall/).
    5.  Save your changes.
    6.  Create the service in your cluster.

        ```
        kubectl apply -f myloadbalancer.yaml
        ```
        {: pre}

        When your load balancer service is created, a portable public IP address is automatically assigned to the load balancer. If no portable public IP address is available, the creation of the load balancer service fails.
3.  Verify that the load balancer service was created successfully. Replace _&lt;myservice&gt;_ with the name of the load balancer service that you created in the previous step.

    ```
    kubectl describe service <myservice>
    ```
    {: pre}

    **Note:** It might take a few minutes for the load balancer service to be created properly and for the app to be available on the public internet.

    Your CLI output looks similar to the following:

    ```
    Name:                   <myservice>
    Namespace:              default
    Labels:                 <none>
    Selector:               <selectorkey>=<selectorvalue>
    Type:                   LoadBalancer
    IP:                     10.10.10.90
    LoadBalancer Ingress:   192.168.10.38
    Port:                   <unset> 8080/TCP
    NodePort:               <unset> 32040/TCP
    Endpoints:              172.30.171.87:8080
    Session Affinity:       None
    Events:
    FirstSeen LastSeen Count From   SubObjectPath Type  Reason   Message
      --------- -------- ----- ----   ------------- -------- ------   -------
      10s  10s  1 {service-controller }   Normal  CreatingLoadBalancer Creating load balancer
      10s  10s  1 {service-controller }   Normal  CreatedLoadBalancer Created load balancer
    ```
    {: screen}

    The **LoadBalancer Ingress** IP address is the portable public IP address that was assigned to your load balancer service.
4.  Access your app from the internet.
    1.  Open your preferred web browser.
    2.  Enter the portable public IP address of the load balancer and port. In the example above, the portable public IP address `192.168.10.38` was assigned to the load balancer service.

        ```
        http://192.168.10.38:8080
        ```
        {: codeblock}


### Configuring public access to an app by using the Ingress controller
{: #cs_apps_public_ingress}

Expose multiple apps in your cluster by creating Ingress resources that are managed by the IBM-provided Ingress controller. The Ingress controller is an external HTTP or HTTPS load balancer that uses a secured and unique public entrypoint to route incoming requests to your apps inside or outside your cluster.

**Note:** Ingress is available for standard clusters only and requires at least two worker nodes in the cluster to ensure high availability.

When you create a standard cluster, an Ingress controller is automatically created for you and assigned a portable public IP address and a public route. You can configure the Ingress controller and define individual routing rules for every app that you expose to the public. Every app that is exposed via Ingress is assigned a unique path that is appended to the public route, so that you can use a unique URL to access an app publicly in your cluster.

When a {{site.data.keyword.Bluemix_notm}} Dedicated account is [enabled for clusters](cs_ov.html#setup_dedicated), you can request public subnets to be used for Ingress controller IP addresses. Then, the Ingress controller is created and a public route is assigned. [Open a support ticket](/docs/support/index.html#contacting-support) to create the subnet, and then use the [`bx cs cluster-subnet-add`](cs_cli_reference.html#cs_cluster_subnet_add) command to add the subnet to the cluster.

You can configure the Ingress controller for the following scenarios.

-   [Use the IBM-provided domain without TLS termination](#ibm_domain)
-   [Use the IBM-provided domain with TLS termination](#ibm_domain_cert)
-   [Use a custom domain and TLS certificate to do TLS termination](#custom_domain_cert)
-   [Use the IBM-provided or a custom domain with TLS termination to access apps outside your cluster](#external_endpoint)
-   [Customize your Ingress controller with annotations](#ingress_annotation)

#### Using the IBM-provided domain without TLS termination
{: #ibm_domain}

You can configure the Ingress controller as an HTTP load balancer for the apps in your cluster and use the IBM-provided domain to access your apps from the internet.

Before you begin:

-   If you do not have one already, [create a standard cluster](cs_cluster.html#cs_cluster_ui).
-   [Target your CLI](cs_cli_install.html#cs_cli_configure) to your cluster to run `kubectl` commands.

To configure the Ingress controller:

1.  [Deploy your app to the cluster](#cs_apps_cli). When you deploy your app to the cluster, one or more pods are created for you that run your app in a container. Ensure that you add a label to your deployment in the metadata section of your configuration script. This label is needed to identify all pods where your app is running, so that they can be included in the Ingress load balancing.
2.  Create a Kubernetes service for the app to expose. The Ingress controller can include your app into the Ingress load balancing only if your app is exposed via a Kubernetes service inside the cluster.
    1.  Open your preferred editor and create a service configuration script that is named, for example, `myservice.yaml`.
    2.  Define a service for the app that you want to expose to the public.

        ```
        apiVersion: v1
        kind: Service
        metadata:
          name: <myservice>
        spec:
          selector:
            <selectorkey>: <selectorvalue>
          ports:
           - protocol: TCP
             port: 8080
        ```
        {: codeblock}

        <table>
        <thead>
        <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
        </thead>
        <tbody>
        <tr>
        <td><code>name</code></td>
        <td>Replace <em>&lt;myservice&gt;</em> with a name for your load balancer service.</td>
        </tr>
        <tr>
        <td><code>selector</code></td>
        <td>Enter the label key (<em>&lt;selectorkey&gt;</em>) and value (<em>&lt;selectorvalue&gt;</em>) pair that you want to use to target the pods where your app runs. For example, if you use the following selector <code>app: code</code>, all pods that have this label in their metadata are included in the load balancing. Enter the same label that you used when you deployed your app to the cluster. </td>
         </tr>
         <tr>
         <td><code>port</code></td>
         <td>The port that the service listens on.</td>
         </tr>
         </tbody></table>
    3.  Save your changes.
    4.  Create the service in your cluster.

        ```
        kubectl apply -f myservice.yaml
        ```
        {: pre}
    5.  Repeat these steps for every app that you want to expose to the public.
3.  Get the details for your cluster to view the IBM-provided domain. Replace _&lt;mycluster&gt;_ with the name of the cluster where the app is deployed that you want to expose to the public.

    ```
    bx cs cluster-get <mycluster>
    ```
    {: pre}

    Your CLI output looks similar to the following.

    ```
    Retrieving cluster <mycluster>...
    OK
    Name:    <mycluster>
    ID:    b9c6b00dc0aa487f97123440b4895f2d
    Created:  2017-04-26T19:47:08+0000
    State:    normal
    Master URL:  https://169.57.40.165:1931
    Ingress subdomain:  <ibmdomain>
    Ingress secret:  <ibmtlssecret>
    Workers:  3
    ```
    {: screen}

    You can see the IBM-provided domain in the **Ingress subdomain** field.
4.  Create an Ingress resource. Ingress resources define the routing rules for the Kubernetes service that you created for your app and are used by the Ingress controller to route incoming network traffic to the service. You can use one Ingress resource to define routing rules for multiple apps as long as every app is exposed via a Kubernetes service inside the cluster.
    1.  Open your preferred editor and create an Ingress configuration script that is named, for example, `myingress.yaml`.
    2.  Define an Ingress resource in your configuration script that uses the IBM-provided domain to route incoming network traffic to the service that you created earlier.

        ```
        apiVersion: extensions/v1beta1
        kind: Ingress
        metadata:
          name: <myingressname>
        spec:
          rules:
          - host: <ibmdomain>
            http:
              paths:
              - path: /<myservicepath1>
                backend:
                  serviceName: <myservice1>
                  servicePort: 80
              - path: /<myservicepath2>
                backend:
                  serviceName: <myservice2>
                  servicePort: 80
        ```
        {: codeblock}

        <table>
        <thead>
        <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
        </thead>
        <tbody>
        <tr>
        <td><code>name</code></td>
        <td>Replace <em>&lt;myingressname&gt;</em> with a name for your Ingress resource.</td>
        </tr>
        <tr>
        <td><code>host</code></td>
        <td>Replace <em>&lt;ibmdomain&gt;</em> with the IBM-provided <strong>Ingress subdomain</strong> name from the previous step.

        </br></br>
        <strong>Note:</strong> Do not use * for your host or leave the host property empty to avoid failures during Ingress creation.</td>
        </tr>
        <tr>
        <td><code>path</code></td>
        <td>Replace <em>&lt;myservicepath1&gt;</em> with a slash or the unique path that your app is listening on, so that network traffic can be forwarded to the app.

        </br>
        For every Kubernetes service, you can define an individual path that is appended to the IBM-provided domain to create a unique path to your app, for example <code>ingress_domain/myservicepath1</code>. When you enter this route into a web browser, network traffic is routed to the Ingress controller. The Ingress controller looks up the associated service, and sends network traffic to the service and to the pods where the app is running by using the same path. The app must be set up to listen on this path in order to receive incoming network traffic.

        </br></br>
        Many apps do not listen on a specific path, but use the root path and a specific port. In this case, define the root path as <code>/</code> and do not specify an individual path for your app.
        </br>
        Examples: <ul><li>For <code>http://ingress_host_name/</code>, enter <code>/</code> as the path.</li><li>For <code>http://ingress_host_name/myservicepath</code>, enter <code>/myservicepath</code> as the path.</li></ul>
        </br>
        <strong>Tip:</strong> If you want to configure your Ingress to listen on a path that is different to the one that your app listens on, you can use the <a href="#rewrite" target="_blank">rewrite annotation</a> to establish proper routing to your app.</td>
        </tr>
        <tr>
        <td><code>serviceName</code></td>
        <td>Replace <em>&lt;myservice1&gt;</em> with the name of the service that you used when you created the Kubernetes service for your app.</td>
        </tr>
        <tr>
        <td><code>servicePort</code></td>
        <td>The port that your service listens to. Use the same port that you defined when you created the Kubernetes service for your app.</td>
        </tr>
        </tbody></table>

    3.  Create the Ingress resource for your cluster.

        ```
        kubectl apply -f myingress.yaml
        ```
        {: pre}

5.  Verify that the Ingress resource was created successfully. Replace _&lt;myingressname&gt;_ with the name of the Ingress resource that you created earlier.

    ```
    kubectl describe ingress <myingressname>
    ```
    {: pre}

  **Note:** It might take a few minutes for the Ingress resource to be created and for the app to be available on the public internet.
6.  In a web browser, enter the URL of the app service to access.

    ```
    http://<ibmdomain>/<myservicepath1>
    ```
    {: codeblock}


#### Using the IBM-provided domain with TLS termination
{: #ibm_domain_cert}

You can configure the Ingress controller to manage incoming TLS connections for your apps, decrypt the network traffic by using the IBM-provided TLS certificate, and forward the unencrypted request to the apps that are exposed in your cluster.

Before you begin:

-   If you do not have one already, [create a standard cluster](cs_cluster.html#cs_cluster_ui).
-   [Target your CLI](cs_cli_install.html#cs_cli_configure) to your cluster to run `kubectl` commands.

To configure the Ingress controller:

1.  [Deploy your app to the cluster](#cs_apps_cli). Ensure that you add a label to your deployment in the metadata section of your configuration script. This label identifies all pods where your app is running, so that the pods are included in the Ingress load balancing.
2.  Create a Kubernetes service for the app to expose. The Ingress controller can include your app into the Ingress load balancing only if your app is exposed via a Kubernetes service inside the cluster.
    1.  Open your preferred editor and create a service configuration script that is named, for example, `myservice.yaml`.
    2.  Define a service for the app that you want to expose to the public.

        ```
        apiVersion: v1
        kind: Service
        metadata:
          name: <myservice>
        spec:
          selector:
            <selectorkey>: <selectorvalue>
          ports:
           - protocol: TCP
             port: 8080
        ```
        {: codeblock}

        <table>
        <thead>
        <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
        </thead>
        <tbody>
        <tr>
        <td><code>name</code></td>
        <td>Replace <em>&lt;myservice&gt;</em> with a name for your Kubernetes service.</td>
        </tr>
        <tr>
        <td><code>selector</code></td>
        <td>Enter the label key (<em>&lt;selectorkey&gt;</em>) and value (<em>&lt;selectorvalue&gt;</em>) pair that you want to use to target the pods where your app runs. For example, if you use the following selector <code>app: code</code>, all pods that have this label in their metadata are included in the load balancing. Enter the same label that you used when you deployed your app to the cluster. </td>
         </tr>
         <tr>
         <td><code>port</code></td>
         <td>The port that the service listens on.</td>
         </tr>
         </tbody></table>

    3.  Save your changes.
    4.  Create the service in your cluster.

        ```
        kubectl apply -f myservice.yaml
        ```
        {: pre}

    5.  Repeat these steps for every app that you want to expose to the public.

3.  View the IBM-provided domain and TLS certificate. Replace _&lt;mycluster&gt;_ with the name of the cluster where the app is deployed.

    ```
    bx cs cluster-get <mycluster>
    ```
    {: pre}

    Your CLI output looks similar to the following.

    ```
    bx cs cluster-get <mycluster>
    Retrieving cluster <mycluster>...
    OK
    Name:    <mycluster>
    ID:    b9c6b00dc0aa487f97123440b4895f2d
    Created:  2017-04-26T19:47:08+0000
    State:    normal
    Master URL:  https://169.57.40.165:1931
    Ingress subdomain:  <ibmdomain>
    Ingress secret:  <ibmtlssecret>
    Workers:  3
    ```
    {: screen}

    You can see the IBM-provided domain in the **Ingress subdomain** and the IBM-provided certificate in the **Ingress secret** field.

4.  Create an Ingress resource. Ingress resources define the routing rules for the Kubernetes service that you created for your app and are used by the Ingress controller to route incoming network traffic to the service. You can use one Ingress resource to define routing rules for multiple apps as long as every app is exposed via a Kubernetes service inside the cluster.
    1.  Open your preferred editor and create an Ingress configuration script that is named, for example, `myingress.yaml`.
    2.  Define an Ingress resource in your configuration script that uses the IBM-provided domain to route incoming network traffic to your services, and the IBM-provided certificate to manage the TLS termination for you. For every service you can define an individual path that is appended to the IBM-provided domain to create a unique path to your app, for example `https://ingress_domain/myapp`. When you enter this route into a web browser, network traffic is routed to the Ingress controller. The Ingress controller looks up the associated service and sends network traffic to the service, and further to the pods where the app is running.

        **Note:** Your app must listen on the path that you defined in the Ingress resource. Otherwise, network traffic cannot be forwarded to the app. Most apps do not listen on a specific path, but use the root path and a specific port. In this case, define the root path as `/` and do not specify an individual path for your app.

        ```
        apiVersion: extensions/v1beta1
        kind: Ingress
        metadata:
          name: <myingressname>
        spec:
          tls:
          - hosts:
            - <ibmdomain>
            secretName: <ibmtlssecret>
          rules:
          - host: <ibmdomain>
            http:
              paths:
              - path: /<myservicepath1>
                backend:
                  serviceName: <myservice1>
                  servicePort: 80
              - path: /<myservicepath2>
                backend:
                  serviceName: <myservice2>
                  servicePort: 80
        ```
        {: codeblock}

        <table>
        <thead>
        <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
        </thead>
        <tbody>
        <tr>
        <td><code>name</code></td>
        <td>Replace <em>&lt;myingressname&gt;</em> with a name for your Ingress resource.</td>
        </tr>
        <tr>
        <td><code>tls/hosts</code></td>
        <td>Replace <em>&lt;ibmdomain&gt;</em> with the IBM-provided <strong>Ingress subdomain</strong> name from the previous step. This domain is configured for TLS termination.

        </br></br>
        <strong>Note:</strong> Do not use &ast; for your host or leave the host property empty to avoid failures during Ingress creation.</td>
        </tr>
        <tr>
        <td><code>tls/secretName</code></td>
        <td>Replace <em>&lt;ibmtlssecret&gt;</em> with the IBM-provided <strong>Ingress secret</strong> name from the previous step. This certificate manages the TLS termination.
        </tr>
        <tr>
        <td><code>host</code></td>
        <td>Replace <em>&lt;ibmdomain&gt;</em> with the IBM-provided <strong>Ingress subdomain</strong> name from the previous step. This domain is configured for TLS termination.

        </br></br>
        <strong>Note:</strong> Do not use &ast; for your host or leave the host property empty to avoid failures during Ingress creation.</td>
        </tr>
        <tr>
        <td><code>path</code></td>
        <td>Replace <em>&lt;myservicepath1&gt;</em> with a slash or the unique path that your app is listening on, so that network traffic can be forwarded to the app.

        </br>
        For every Kubernetes service, you can define an individual path that is appended to the IBM-provided domain to create a unique path to your app, for example <code>ingress_domain/myservicepath1</code>. When you enter this route into a web browser, network traffic is routed to the Ingress controller. The Ingress controller looks up the associated service, and sends network traffic to the service and to the pods where the app is running by using the same path. The app must be set up to listen on this path in order to receive incoming network traffic.

        </br>
        Many apps do not listen on a specific path, but use the root path and a specific port. In this case, define the root path as <code>/</code> and do not specify an individual path for your app.

        </br>
        Examples: <ul><li>For <code>http://ingress_host_name/</code>, enter <code>/</code> as the path.</li><li>For <code>http://ingress_host_name/myservicepath</code>, enter <code>/myservicepath</code> as the path.</li></ul>
        <strong>Tip:</strong> If you want to configure your Ingress to listen on a path that is different to the one that your app listens on, you can use the <a href="#rewrite" target="_blank">rewrite annotation</a> to establish proper routing to your app.</td>
        </tr>
        <tr>
        <td><code>serviceName</code></td>
        <td>Replace <em>&lt;myservice1&gt;</em> with the name of the service that you used when you created the Kubernetes service for your app.</td>
        </tr>
        <tr>
        <td><code>servicePort</code></td>
        <td>The port that your service listens to. Use the same port that you defined when you created the Kubernetes service for your app.</td>
        </tr>
        </tbody></table>

    3.  Create the Ingress resource for your cluster.

        ```
        kubectl apply -f myingress.yaml
        ```
        {: pre}

5.  Verify that the Ingress resource was created successfully. Replace _&lt;myingressname&gt;_ with the name of the Ingress resource that you created earlier.

    ```
    kubectl describe ingress <myingressname>
    ```
    {: pre}

    **Note:** It might take a few minutes for the Ingress resource to be created properly and for the app to be available on the public internet.
6.  In a web browser, enter the URL of the app service to access.

    ```
    https://<ibmdomain>/<myservicepath1>
    ```
    {: codeblock}

#### Using the Ingress controller with a custom domain and TLS certificate
{: #custom_domain_cert}

You can configure the Ingress controller to route incoming network traffic to the apps in your cluster and use your own TLS certificate to manage the TLS termination, while using your custom domain rather than the IBM-provided domain.
{:shortdesc}

Before you begin:

-   If you do not have one already, [create a standard cluster](cs_cluster.html#cs_cluster_ui).
-   [Target your CLI](cs_cli_install.html#cs_cli_configure) to your cluster to run `kubectl` commands.

To configure the Ingress controller:

1.  Create a custom domain. To create a custom domain, work with your Domain Name Service (DNS) provider to register your custom domain.
2.  Configure your domain to route incoming network traffic to the IBM Ingress controller. Choose between these options:
    -   Define an alias for your custom domain by specifying the IBM-provided domain as a Canonical Name record (CNAME). To find the IBM-provided Ingress domain, run `bx cs cluster-get <mycluster>` and look for the **Ingress subdomain** field.
    -   Map your custom domain to the portable public IP address of the IBM-provided Ingress controller by adding the IP address as a Pointer record (PTR). To find the portable public IP address of the Ingress controller:
        1.  Run `bx cs cluster-get <mycluster>` and look for the **Ingress subdomain** field.
        2.  Run `nslookup <Ingress subdomain>`.
3.  Create a TLS certificate and key for your domain that is encoded in base64 format.
4.  Store your TLS certificate and key in a Kubernetes secret.
    1.  Open your preferred editor and create a Kubernetes secret configuration script that is named, for example, `mysecret.yaml`.
    2.  Define a secret that uses your TLS certificate and key.

        ```
        apiVersion: v1
        kind: Secret
        metadata:
          name: <mytlssecret>
        type: Opaque
        data:
          tls.crt: <tlscert>
          tls.key: <tlskey>
        ```
        {: codeblock}

        <table>
        <thead>
        <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
        </thead>
        <tbody>
        <tr>
        <td><code>name</code></td>
        <td>Replace <em>&lt;mytlssecret&gt;</em> with a name for your Kubernetes secret.</td>
        </tr>
        <tr>
        <td><code>tls.cert</code></td>
        <td>Replace <em>&lt;tlscert&gt;</em> with your custom TLS certificate that is encoded in base64 format.</td>
         </tr>
         <td><code>tls.key</code></td>
         <td>Replace <em>&lt;tlskey&gt;</em> with your custom TLS key that is encoded in base64 format.</td>
         </tbody></table>

    3.  Save your configuration script.
    4.  Create the TLS secret for your cluster.

        ```
        kubectl apply -f mysecret.yaml
        ```
        {: pre}

5.  [Deploy your app to the cluster](#cs_apps_cli). When you deploy your app to the cluster, one or more pods are created for you that run your app in a container. Enure that you add a label to your deployment in the metadata section of your configuration script. This label is needed to identify all pods where your app is running, so that they can be included in the Ingress load balancing.

6.  Create a Kubernetes service for the app to expose. The Ingress controller can include your app into the Ingress load balancing only if your app is exposed via a Kubernetes service inside the cluster.

    1.  Open your preferred editor and create a service configuration script that is named, for example, `myservice.yaml`.
    2.  Define a service for the app that you want to expose to the public.

        ```
        apiVersion: v1
        kind: Service
        metadata:
          name: <myservice>
        spec:
          selector:
            <selectorkey>: <selectorvalue>
          ports:
           - protocol: TCP
             port: 8080
        ```
       {: codeblock}

        <table>
        <thead>
        <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
        </thead>
        <tbody>
        <tr>
        <td><code>name</code></td>
        <td>Replace <em>&lt;myservice1&gt;</em> with a name for your Kubernetes service.</td>
        </tr>
        <tr>
        <td><code>selector</code></td>
        <td>Enter the label key (<em>&lt;selectorkey&gt;</em>) and value (<em>&lt;selectorvalue&gt;</em>) pair that you want to use to target the pods where your app runs. For example, if you use the following selector <code>app: code</code>, all pods that have this label in their metadata are included in the load balancing. Enter the same label that you used when you deployed your app to the cluster. </td>
         </tr>
         <td><code>port</code></td>
         <td>The port that the service listens on.</td>
         </tbody></table>

    3.  Save your changes.
    4.  Create the service in your cluster.

        ```
        kubectl apply -f myservice.yaml
        ```
        {: pre}

    5.  Repeat these steps for every app that you want to expose to the public.
7.  Create an Ingress resource. Ingress resources define the routing rules for the Kubernetes service that you created for your app and are used by the Ingress controller to route incoming network traffic to the service. You can use one Ingress resource to define routing rules for multiple apps as long as every app is exposed via a Kubernetes service inside the cluster.
    1.  Open your preferred editor and create an Ingress configuration script that is named, for example, `myingress.yaml`.
    2.  Define an Ingress resource in your configuration script that uses your custom domain to route incoming network traffic to your services, and your custom certificate to manage the TLS termination. For every service you can define an individual path that is appended to your custom domain to create a unique path to your app, for example `https://mydomain/myapp`. When you enter this route into a web browser, network traffic is routed to the Ingress controller. The Ingress controller looks up the associated service and sends network traffic to the service, and further to the pods where the app is running.

        **Note:** It is important that the app listens on the path that you defined in the Ingress resource. Otherwise, network traffic cannot be forwarded to the app. Most apps do not listen on a specific path, but use the root path and a specific port. In this case, define the root path as `/` and do not specify an individual path for your app.

        ```
        apiVersion: extensions/v1beta1
        kind: Ingress
        metadata:
          name: <myingressname>
        spec:
          tls:
          - hosts:
            - <mycustomdomain>
            secretName: <mytlssecret>
          rules:
          - host: <mycustomdomain>
            http:
              paths:
              - path: /<myservicepath1>
                backend:
                  serviceName: <myservice1>
                  servicePort: 80
              - path: /<myservicepath2>
                backend:
                  serviceName: <myservice2>
                  servicePort: 80
        ```
        {: codeblock}

        <table>
        <thead>
        <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
        </thead>
        <tbody>
        <tr>
        <td><code>name</code></td>
        <td>Replace <em>&lt;myingressname&gt;</em> with a name for your Ingress resource.</td>
        </tr>
        <tr>
        <td><code>tls/hosts</code></td>
        <td>Replace <em>&lt;mycustomdomain&gt;</em> with our custom domain that you want to configure for TLS termination.

        </br></br>
        <strong>Note:</strong> Do not use &ast; for your host or leave the host property empty to avoid failures during Ingress creation.</td>
        </tr>
        <tr>
        <td><code>tls/secretName</code></td>
        <td>Replace <em>&lt;mytlssecret&gt;</em> with the name of the secret that you created earlier and that holds your custom TLS certificate and key to manage the TLS termination for your custom domain.
        </tr>
        <tr>
        <td><code>host</code></td>
        <td>Replace <em>&lt;mycustomdomain&gt;</em> with our custom domain that you want to configure for TLS termination.

        </br></br>
        <strong>Note:</strong> Do not use &ast; for your host or leave the host property empty to avoid failures during Ingress creation.
        </td>
        </tr>
        <tr>
        <td><code>path</code></td>
        <td>Replace <em>&lt;myservicepath1&gt;</em> with a slash or the unique path that your app is listening on, so that network traffic can be forwarded to the app.

        </br>
        For every Kubernetes service, you can define an individual path that is appended to the IBM-provided domain to create a unique path to your app, for example <code>ingress_domain/myservicepath1</code>. When you enter this route into a web browser, network traffic is routed to the Ingress controller. The Ingress controller looks up the associated service, and sends network traffic to the service and to the pods where the app is running by using the same path. The app must be set up to listen on this path in order to receive incoming network traffic.

        </br>
        Many apps do not listen on a specific path, but use the root path and a specific port. In this case, define the root path as <code>/</code> and do not specify an individual path for your app.

        </br></br>
        Examples: <ul><li>For <code>https://mycustomdomain/</code>, enter <code>/</code> as the path.</li><li>For <code>https://mycustomdomain/myservicepath</code>, enter <code>/myservicepath</code> as the path.</li></ul>
        <strong>Tip:</strong> If you want to configure your Ingress to listen on a path that is different to the one that your app listens on, you can use the <a href="#rewrite" target="_blank">rewrite annotation</a> to establish proper routing to your app.
        </td>
        </tr>
        <tr>
        <td><code>serviceName</code></td>
        <td>Replace <em>&lt;myservice1&gt;</em> with the name of the service that you used when you created the Kubernetes service for your app.</td>
        </tr>
        <tr>
        <td><code>servicePort</code></td>
        <td>The port that your service listens to. Use the same port that you defined when you created the Kubernetes service for your app.</td>
        </tr>
        </tbody></table>

    3.  Save your changes.
    4.  Create the Ingress resource for your cluster.

        ```
        kubectl apply -f myingress.yaml
        ```
        {: pre}

8.  Verify that the Ingress resource was created successfully. Replace _&lt;myingressname&gt;_ with the name of the Ingress resource that you created earlier.

    ```
    kubectl describe ingress <myingressname>
    ```
    {: pre}

    **Note:** It might take a few minutes for the Ingress resource to be created properly and for the app to be available on the public internet.

9.  Access your app from the internet.
    1.  Open your preferred web browser.
    2.  Enter the URL of the app service that you want to access.

        ```
        https://<mycustomdomain>/<myservicepath1>
        ```
        {: codeblock}


#### Configuring the Ingress controller to route network traffic to apps outside the cluster
{: #external_endpoint}

You can configure the Ingress controller for apps that are located outside of the cluster to be included into the cluster load balancing. Incoming requests on the IBM-provided or your custom domain are forwarded automatically to the external app.

Before you begin:

-   If you do not have one already, [create a standard cluster](cs_cluster.html#cs_cluster_ui).
-   [Target your CLI](cs_cli_install.html#cs_cli_configure) to your cluster to run `kubectl` commands.
-   Ensure that the external app that you want to include into the cluster load balancing can be accessed by using a public IP address.

You can configure the Ingress controller to route incoming network traffic on the IBM-provided domain to apps that are located outside your cluster. If you want to use a custom domain and TLS certificate instead, replace the IBM-provided domain and TLS certificate with your [custom domain and TLS certificate](#custom_domain_cert).

1.  Configure a Kubernetes endpoint that defines the external location of the app that you want to include into the cluster load balancing.
    1.  Open your preferred editor and create an endpoint configuration script that is named, for example, `myexternalendpoint.yaml`.
    2.  Define your external endpoint. Include all public IP addresses and ports that you can use to access your external app.

        ```
        kind: Endpoints
        apiVersion: v1
        metadata:
          name: <myendpointname>
        subsets:
          - addresses:
              - ip: <externalIP1>
              - ip: <externalIP2>
            ports:
              - port: <externalport>
        ```
        {: codeblock}

        <table>
        <thead>
        <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
        </thead>
        <tbody>
        <tr>
        <td><code>name</code></td>
        <td>Replace <em>&lt;myendpointname&gt;</em> with the name for your Kubernetes endpoint.</td>
        </tr>
        <tr>
        <td><code>ip</code></td>
        <td>Replace <em>&lt;externalIP&gt;</em> with the public IP addresses to connect to your external app.</td>
         </tr>
         <td><code>port</code></td>
         <td>Replace <em>&lt;externalport&gt;</em> with the port that your external app listens to.</td>
         </tbody></table>

    3.  Save your changes.
    4.  Create the Kubernetes endpoint for your cluster.

        ```
        kubectl apply -f myexternalendpoint.yaml
        ```
        {: pre}

2.  Create a Kubernetes service for your cluster and configure it to forward incoming requests to the external endpoint that you created earlier.
    1.  Open your preferred editor and create a service configuration script that is named, for example, `myexternalservice.yaml`.
    2.  Define the service.

        ```
        apiVersion: v1
        kind: Service
        metadata:
          name: <myexternalservice>
          labels:
              name: <myendpointname>
        spec:
          ports:
           - protocol: TCP
             port: 8080
        ```
        {: codeblock}

        <table>
        <thead>
        <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
        </thead>
        <tbody>
        <tr>
        <td><code>metadata/name</code></td>
        <td>Replace <em>&lt;myexternalservice&gt;</em> with the name for your Kubernetes service.</td>
        </tr>
        <tr>
        <td><code>labels/name</code></td>
        <td>Replace <em>&lt;myendpointname&gt;</em> with the name of the Kubernetes endpoint that you created earlier.</td>
        </tr>
        <tr>
        <td><code>port</code></td>
        <td>The port that the service listens on.</td>
        </tr></tbody></table>

    3.  Save your changes.
    4.  Create the Kubernetes service for your cluster.

        ```
        kubectl apply -f myexternalservice.yaml
        ```
        {: pre}

3.  View the IBM-provided domain and TLS certificate. Replace _&lt;mycluster&gt;_ with the name of the cluster where the app is deployed.

    ```
    bx cs cluster-get <mycluster>
    ```
    {: pre}

    Your CLI output looks similar to the following.

    ```
    Retrieving cluster <mycluster>...
    OK
    Name:    <mycluster>
    ID:    b9c6b00dc0aa487f97123440b4895f2d
    Created:  2017-04-26T19:47:08+0000
    State:    normal
    Master URL:  https://169.57.40.165:1931
    Ingress subdomain:  <ibmdomain>
    Ingress secret:  <ibmtlssecret>
    Workers:  3
    ```
    {: screen}

    You can see the IBM-provided domain in the **Ingress subdomain** and the IBM-provided certificate in the **Ingress secret** field.

4.  Create an Ingress resource. Ingress resources define the routing rules for the Kubernetes service that you created for your app and are used by the Ingress controller to route incoming network traffic to the service. You can use one Ingress resource to define routing rules for multiple external apps as long as every app is exposed with its external endpoint via a Kubernetes service inside the cluster.
    1.  Open your preferred editor and create an Ingress configuration script that is named, for example, `myexternalingress.yaml`.
    2.  Define an Ingress resource in your configuration script that uses the IBM-provided domain and TLS certificate to route incoming network traffic to your external app by using the external endpoint that you defined earlier. For every service you can define an individual path that is appended to the IBM-provided or custom domain to create a unique path to your app, for example `https://ingress_domain/myapp`. When you enter this route into a web browser, network traffic is routed to the Ingress controller. The Ingress controller looks up the associated service and sends network traffic to the service, and further to the external app.

        **Note:** It is important that the app listens on the path that you defined in the Ingress resource. Otherwise, network traffic cannot be forwarded to the app. Most apps do not listen on a specific path, but use the root path and a specific port. In this case, define the root path as / and do not specify an individual path for your app.

        ```
        apiVersion: extensions/v1beta1
        kind: Ingress
        metadata:
          name: <myingressname>
        spec:
          tls:
          - hosts:
            - <ibmdomain>
            secretName: <ibmtlssecret>
          rules:
          - host: <ibmdomain>
            http:
              paths:
              - path: /<myexternalservicepath1>
                backend:
                  serviceName: <myservice1>
                  servicePort: 80
              - path: /<myexternalservicepath2>
                backend:
                  serviceName: <myexternalservice2>
                  servicePort: 80
        ```
        {: codeblock}

        <table>
        <thead>
        <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
        </thead>
        <tbody>
        <tr>
        <td><code>name</code></td>
        <td>Replace <em>&lt;myingressname&gt;</em> with the name for the Ingress resource.</td>
        </tr>
        <tr>
        <td><code>tls/hosts</code></td>
        <td>Replace <em>&lt;ibmdomain&gt;</em> with the IBM-provided <strong>Ingress subdomain</strong> name from the previous step. This domain is configured for TLS termination.

        </br></br>
        <strong>Note:</strong> Do not use &ast; for your host or leave the host property empty to avoid failures during Ingress creation.</td>
        </tr>
        <tr>
        <td><code>tls/secretName</code></td>
        <td>Replace <em>&lt;ibmtlssecret&gt;</em> with the IBM-provided <strong>Ingress secret</strong> from the previous step. This certificate manages the TLS termination.</td>
        </tr>
        <tr>
        <td><code>rules/host</code></td>
        <td>Replace <em>&lt;ibmdomain&gt;</em> with the IBM-provided <strong>Ingress subdomain</strong> name from the previous step. This domain is configured for TLS termination.

        </br></br>
        <strong>Note:</strong> Do not use &ast; for your host or leave the host property empty to avoid failures during Ingress creation.</td>
        </tr>
        <tr>
        <td><code>path</code></td>
        <td>Replace <em>&lt;myexternalservicepath&gt;</em> with a slash or the unique path that your external app is listening on, so that network traffic can be forwarded to the app.

        </br>
        For every Kubernetes service, you can define an individual path that is appended to your domain to create a unique path to your app, for example <code>https://ibmdomain/myservicepath1</code>. When you enter this route into a web browser, network traffic is routed to the Ingress controller. The Ingress controller looks up the associated service, and sends network traffic to the external app by using the same path. The app must be set up to listen on this path in order to receive incoming network traffic.

        </br></br>
        Many apps do not listen on a specific path, but use the root path and a specific port. In this case, define the root path as <code>/</code> and do not specify an individual path for your app.

        </br></br>
        <strong>Tip:</strong> If you want to configure your Ingress to listen on a path that is different to the one that your app listens on, you can use the <a href="#rewrite" target="_blank">rewrite annotation</a> to establish proper routing to your app.</td>
        </tr>
        <tr>
        <td><code>serviceName</code></td>
        <td>Replace <em>&lt;myexternalservice&gt;</em> with the name of the service that you used when you created the Kubernetes service for your external app.</td>
        </tr>
        <tr>
        <td><code>servicePort</code></td>
        <td>The port that your service listens to.</td>
        </tr>
        </tbody></table>

    3.  Save your changes.
    4.  Create the Ingress resource for your cluster.

        ```
        kubectl apply -f myexternalingress.yaml
        ```
        {: pre}

5.  Verify that the Ingress resource was created successfully. Replace _&lt;myingressname&gt;_ with the name of the Ingress resource that you created earlier.

    ```
    kubectl describe ingress <myingressname>
    ```
    {: pre}

    **Note:** It might take a few minutes for the Ingress resource to be created properly and for the app to be available on the public internet.

6.  Access your external app.
    1.  Open your preferred web browser.
    2.  Enter the URL to access your external app.

        ```
        https://<ibmdomain>/<myexternalservicepath>
        ```
        {: codeblock}


#### Supported Ingress annotations
{: #ingress_annotation}

You can specify metadata for your Ingress resource to add capabilities to your Ingress controller.
{: shortdesc}

|Supported annotation|Description|
|--------------------|-----------|
|[Rewrites](#rewrite)|Route incoming network traffic to a different path that your back-end app listens on.|
|[Session-affinity with cookies](#sticky_cookie)|Always route incoming network traffic to the same upstream server by using a sticky cookie.|
|[Additional client request or response header](#add_header)|Add extra header information to a client request before forwarding the request to your back-end app, or to a client response before sending the reponse to the client.|
|[Client response header removal](#remove_response_headers)|Remove header information from a client response before forwarding the response to the client.|
|[HTTP redirects to HTTPs](#redirect_http_to_https)|Redirect insecure HTTP requests on your domain to HTTPs.|
|[Client response data buffering](#response_buffer)|Disable the buffering of a client response on the Ingress controller while sending the response to the client.|
|[Custom connect-timeouts and read-timeouts](#timeout)|Adjust the time the Ingress controller waits to connect to and read from the back-end app before the back-end app is considered to be not available.|
|[Custom maximum client request body size](#client_max_body_size)|Adjust the size of the client request body that is allowed to be sent to the Ingress controller.|

##### **Route incoming network traffic to a different path by using rewrites**
{: #rewrite}

Use the rewrite annotation to route incoming network traffic on an Ingress controller domain path to a different path that your back-end application listens on.
{:shortdesc}

<dl>
<dt>Description</dt>
<dd>Your Ingress controller domain routes incoming network traffic on <code>mykubecluster.us-south.containers.mybluemix.net/beans</code> to your app. Your app listens on <code>/coffee</code>, instead of <code>/beans</code>. To forward incoming network traffic to your app, add the rewrite annotation to your Ingress resource configuration file, so that incoming network traffic on <code>/beans</code> is forwarded to your app by using the <code>/coffee</code> path. When including multiple services, use only a semi-colon (;) to separate them.</dd>
<dt>Sample Ingress resource YAML</dt>
<dd>
<pre class="codeblock">
<code>apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myingress
  annotations:
    ingress.bluemix.net/rewrite-path: "serviceName=&lt;service_name1&gt; rewrite=&lt;rewrite_path1&gt;;serviceName=&lt;service_name2&gt; rewrite=&lt;rewrite_path2&gt;"
spec:
  tls:
  - hosts:
    - mydomain
    secretName: mytlssecret
  rules:
  - host: mydomain
    http:
      paths:
      - path: /&lt;domain_path1&gt;
        backend:
          serviceName: &lt;service_name1&gt;
          servicePort: &lt;service_port1&gt;
      - path: /&lt;domain_path2&gt;
        backend:
          serviceName: &lt;service_name2&gt;
          servicePort: &lt;service_port2&gt;</code></pre>

<table>
<thead>
<th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
</thead>
<tbody>
<tr>
<td><code>annotations</code></td>
<td>Replace <em>&lt;service_name&gt;</em> with the name of the Kubernetes service that you created for your app, and <em>&lt;rewrite-path&gt;</em> with the path that your app listens on. Incoming network traffic on the Ingress controller domain is forwarded to the Kubernetes service by using this path. Most apps do not listen on a specific path, but use the root path and a specific port. In this case, define <code>/</code> as the <em>&lt;rewrite-path&gt;</em> for your app.</td>
</tr>
<tr>
<td><code>path</code></td>
<td>Replace <em>&lt;domain_path&gt;</em> with the path that you want to append to your Ingress controller domain. Incoming network traffic on this path is forwarded to the rewrite path that you defined in your annotation. In the example above, set the domain path to <code>/beans</code> to include this path into the load balancing of your Ingress controller.</td>
</tr>
<tr>
<td><code>serviceName</code></td>
<td>Replace <em>&lt;service_name&gt;</em> with the name of the Kubernetes service that you created for your app. The service name that you use here must be the same name that you defined in your annotation.</td>
</tr>
<tr>
<td><code>servicePort</code></td>
<td>Replace <em>&lt;service_port&gt;</em> with the port that your service listens on.</td>
</tr></tbody></table>

</dd></dl>


##### **Always route incoming network traffic to the same upstream server by using a sticky cookie**
{: #sticky_cookie}

Use the sticky cookie annotation to add session affinity to your Ingress controller.
{:shortdesc}

<dl>
<dt>Description</dt>
<dd>Your app setup might require you to deploy multiple upstream servers that handle incoming client requests and that ensure higher availability. When a client connects to your back-end app, it might be useful that a client is served by the same upstream server for the duration of a session or for the time it takes to complete a task. You can configure your Ingress controller to ensure session-affinity by always routing incoming network traffic to the same upstream server.

</br>
Every client that connects to your back-end app is assigned to one of the available upstream servers by the Ingress controller. The Ingress controller creates a session cookie that is stored in the client's app and that is included in the header information of every request between the Ingress controller and the client. The information in the cookie ensures that all requests are handled by the same upstream server throughout the session.

</br></br>
When you include multiple services, use a semi-colon (;) to separate them.</dd>
<dt>Sample Ingress resource YAML</dt>
<dd>

<pre class="codeblock">
<code>apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myingress
  annotations:
    ingress.bluemix.net/sticky-cookie-services: "serviceName=&lt;service_name1&gt; name=&lt;cookie_name1&gt; expires=&lt;expiration_time1&gt; path=&lt;cookie_path1&gt; hash=&lt;hash_algorithm1&gt;;serviceName=&lt;service_name2&gt; name=&lt;cookie_name2&gt; expires=&lt;expiration_time2&gt; path=&lt;cookie_path2&gt; hash=&lt;hash_algorithm2&gt;"
spec:
  tls:
  - hosts:
    - mydomain
    secretName: mytlssecret
  rules:
  - host: mydomain
    http:
      paths:
      - path: /
        backend:
          serviceName: &lt;service_name1&gt;
          servicePort: 8080
      - path: /myapp
        backend:
          serviceName: &lt;service_name2&gt;
          servicePort: 80</code></pre>

  <table>
  <caption>Table 12. Understanding the YAML file components</caption>
  <thead>
  <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
  </thead>
  <tbody>
  <tr>
  <td><code>annotations</code></td>
  <td>Replace the following values:<ul><li><code><em>&lt;service_name&gt;</em></code>: The name of the Kubernetes service that you created for your app.</li><li><code><em>&lt;cookie_name&gt;</em></code>: Choose a name of the sticky cookie that is created during a session.</li><li><code><em>&lt;expiration_time&gt;</em></code>: The time in seconds, minutes, or hours before the sticky cookie expires. This time is independent of the user activity. After the cookie is expired, the cookie is deleted by the client web browser and no longer sent to the Ingress controller. For example, to set an expiration time of 1 second, 1 minute, or 1 hour, enter <strong>1s</strong>, <strong>1m</strong>, or <strong>1h</strong>.</li><li><code><em>&lt;cookie_path&gt;</em></code>: The path that is appended to the Ingress subdomain and that indicates for which domains and subdomains the cookie is sent to the Ingress controller. For example, if your Ingress domain is <code>www.myingress.com</code> and you want to send the cookie in every client request, you must set <code>path=/</code>. If you want to send the cookie only for <code>www.myingress.com/myapp</code> and all its subdomains, then you must set <code>path=/myapp</code>.</li><li><code><em>&lt;hash_algorithm&gt;</em></code>: The hash algorithm that protects the information in the cookie. Only <code>sha1</code> is supported. SHA1 creates a hash sum based on the information in the cookie and appends this hash sum to the cookie. The server can decrypt the information in the cookie and verify data integrity.</li></ul></td>
  </tr>
  </tbody></table>

 </dd></dl>


##### **Adding custom http headers to a client request or client response**
{: #add_header}

Use this annotation to add extra header information to a client request before sending the request to the back-end app, or to a client response before sending the response to the client.
{:shortdesc}

<dl>
<dt>Description</dt>
<dd>The Ingress controller acts as a proxy between the client app and your back-end app. Client requests that are sent to the Ingress controller are processed (proxied), and put into a new request that is then sent from the Ingress controller to your back-end app. Proxying a request removes http header information, such as the user name, that was initially sent from the client. If your back-end app requires this information, you can use the <strong>ingress.bluemix.net/proxy-add-headers</strong> annotation to add header information to the client request before the request is forwarded from the Ingress controller to your back-end app.

</br></br>
When a back-end app sends a response to the client, the response is proxied by the Ingress controller and http headers are removed from the response. The client web app might require this header information to successfully process the response. You can use the <strong>ingress.bluemix.net/response-add-headers</strong> annotation to add header information to the client response before the response is forwarded from the Ingress controller to client web app.</dd>
<dt>Sample Ingress resource YAML</dt>
<dd>

<pre class="codeblock">
<code>apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myingress
  annotations:
    ingress.bluemix.net/proxy-add-headers: |
      serviceName=&lt;service_name1&gt; {
      &lt;header1> &lt;value1&gt;;
      &lt;header2> &lt;value2&gt;;
      }
      serviceName=&lt;service_name2&gt; {
      &lt;header3&gt; &lt;value3&gt;;
      }
    ingress.bluemix.net/response-add-headers: |
      serviceName=&lt;service_name1&gt; {
      "&lt;header1&gt;: &lt;value1&gt;";
      "&lt;header2&gt;: &lt;value2&gt;";
      }
      serviceName=&lt;service_name1&gt; {
      "&lt;header3&gt;: &lt;value3&gt;";
      }
spec:
  tls:
  - hosts:
    - mydomain
    secretName: mytlssecret
  rules:
  - host: mydomain
    http:
      paths:
      - path: /
        backend:
          serviceName: &lt;service_name1&gt;
          servicePort: 8080
      - path: /myapp
        backend:
          serviceName: &lt;service_name2&gt;
          servicePort: 80</code></pre>

 <table>
  <thead>
  <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
  </thead>
  <tbody>
  <tr>
  <td><code>annotations</code></td>
  <td>Replace the following values:<ul><li><code><em>&lt;service_name&gt;</em></code>: The name of the Kubernetes service that you created for your app.</li><li><code><em>&lt;header&gt;</em></code>: The key of the header information to add to the client request or client response.</li><li><code><em>&lt;value&gt;</em></code>: The value of the header information to add to the client request or client response.</li></ul></td>
  </tr>
  </tbody></table>

 </dd></dl>

##### **Removing http header information from a client's response**
{: #remove_response_headers}

Use this annotation to remove header information that is included in the client response from the back-end end app before the response is sent to the client.
{:shortdesc}

<dl>
<dt>Description</dt>
<dd>The Ingress controller acts as a proxy between your back-end app and the client web browser. Client responses from the back-end app that are sent to the Ingress controller are processed (proxied), and put into a new response that is then sent from the Ingress controller to the client web browser. Although proxying a response removes http header information that was initially sent from the back-end app, this process might not remove all back-end app specific headers. Use this annotation to remove header information from a client reponse before the response is forwarded from the Ingress controller to the client web browser.</dd>
<dt>Sample Infress resource YAML</dt>
<dd>
<pre class="codeblock">
<code>apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myingress
  annotations:
    ingress.bluemix.net/response-remove-headers: |
      serviceName=&lt;service_name1&gt; {
      "&lt;header1&gt;";
      "&lt;header2&gt;";
      }
      serviceName=&lt;service_name2&gt; {
      "&lt;header3&gt;";
      }
spec:
  tls:
  - hosts:
    - mydomain
    secretName: mytlssecret
  rules:
  - host: mydomain
    http:
      paths:
      - path: /
        backend:
          serviceName: &lt;service_name1&gt;
          servicePort: 8080
      - path: /myapp
        backend:
          serviceName: &lt;service_name2&gt;
          servicePort: 80</code></pre>

 <table>
  <thead>
  <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
  </thead>
  <tbody>
  <tr>
  <td><code>annotations</code></td>
  <td>Replace the following values:<ul><li><code><em>&lt;service_name&gt;</em></code>: The name of the Kubernetes service that you created for your app.</li><li><code><em>&lt;header&gt;</em></code>: The key of the header to remove from the client response.</li></ul></td>
  </tr>
  </tbody></table>

  </dd></dl>


##### **Redirecting insecure http client requests to https**
{: #redirect_http_to_https}

Use the redirect annotation to convert insecure http client requests to https.
{:shortdesc}

<dl>
<dt>Description</dt>
<dd>You set up your Ingress controller to secure your domain with the IBM-provided TLS certificate or your custom TLS certificate. Some users might try to access your apps by using an insecure http request to your Ingress controller domain, for example <code>http://www.myingress.com</code>, instead of using <code>https</code>. You can use the redirect annotation to always convert insecure http requests to https. If you do not use this annotation, insecure http requests are not converted into https requests by default and might expose confidential information to the public without encryption.

</br></br>
Redirecting http requests to https is disabled by default.</dd>
<dt>Sample Ingress resource YAML</dt>

<pre class="codeblock">
<code>apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myingress
  annotations:
    ingress.bluemix.net/redirect-to-https: "True"
spec:
  tls:
  - hosts:
    - mydomain
    secretName: mytlssecret
  rules:
  - host: mydomain
    http:
      paths:
      - path: /
        backend:
          serviceName: myservice
          servicePort: 8080</code></pre>
</dd></dl>

##### **Disabling buffering of back-end responses on your Ingress controller**
{: #response_buffer}

Use the buffer annotation to disable the storage of response data on the Ingress controller while the data is sent to the client.
{:shortdesc}

<dl>
<dt>Description</dt>
<dd>The Ingress controller acts as a proxy between your back-end app and the client web browser. When a response is sent from the back-end app to the client, the response data is buffered on the Ingress controller by default. The Ingress controller proxies the client response and starts sending the response to the client at the client's pace. After all data from the back-end app is received by the Ingress controller, the connection to the back-end app is closed. The connection from the Ingress controller to the client remains open until the client received all data.

</br></br>
If buffering of response data on the Ingress controller is disabled, data is immediately sent from the Ingress controller to the client. The client must be able to handle incoming data at the pace of the Ingress controller. If the client is too slow, data might get lost.
</br></br>
Response data buffering on the Ingress controller is enabled by default.</dd>
<dt>Sample Ingress resource YAML</dt>
<dd>

<pre class="codeblock">
<code>apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myingress
  annotations:
    ingress.bluemix.net/proxy-buffering: "False"
spec:
  tls:
  - hosts:
    - mydomain
    secretName: mytlssecret
  rules:
  - host: mydomain
    http:
      paths:
      - path: /
        backend:
          serviceName: myservice
          servicePort: 8080</code></pre>
</dd></dl>


##### **Setting a custom connect-timeout and read-timeout for the Ingress controller**
{: #timeout}

Adjust the time for the Ingress controller to wait while connecting and reading from the back-end app before the back-end app is considered to be not available.
{:shortdesc}

<dl>
<dt>Description</dt>
<dd>When a client request is sent to the Ingress controller, a connection to the back-end app is opened by the Ingress controller. By default, the Ingress controller waits 60 seconds to receive a reply from the back-end app. If the back-end app does not reply within 60 seconds, then the connection request is aborted and the back-end app is considered to be not available.

</br></br>
After the Ingress controller is connected to the back-end app, response data from the back-end app is read by the Ingress controller. During this read operation, the Ingress controller waits a maximum of 60 seconds between two read operations to receive data from the back-end app. If the back-end app does not send data within 60 seconds, the connection to the back-end app is closed and the app is considered to be not available.
</br></br>
A 60 second connect-timeout and read-timeout is the default timeout on a proxy and ideally should not be changed.
</br></br>
If the availability of your app is not steady or your app is slow to respond because of high workloads, you might want to increase the connect-timeout or read-timeout. Keep in mind that increasing the timeout impacts the performance of the Ingress controller as the connection to the back-end app must stay open until the timeout is reached.
</br></br>
On the other hand, you can decrease the timeout to gain performance on the Ingress controller. Ensure that your back-end app is able to handle requests within the specified timeout, even during higher workloads.</dd>
<dt>Sample Ingress resource YAML</dt>
<dd>

<pre class="codeblock">
<code>apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myingress
  annotations:
    ingress.bluemix.net/proxy-connect-timeout: "&lt;connect_timeout&gt;s"
    ingress.bluemix.net/proxy-read-timeout: "&lt;read_timeout&gt;s"
spec:
  tls:
  - hosts:
    - mydomain
    secretName: mytlssecret
  rules:
  - host: mydomain
    http:
      paths:
      - path: /
        backend:
          serviceName: myservice
          servicePort: 8080</code></pre>

<table>
  <thead>
  <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
  </thead>
  <tbody>
  <tr>
  <td><code>annotations</code></td>
  <td>Replace the following values:<ul><li><code><em>&lt;connect_timeout&gt;</em></code>: Enter the number of seconds to wait to connect to the back-end app, for example <strong>65s</strong>.

  </br></br>
  <strong>Note:</strong> A connect-timeout cannot exceed 75 seconds.</li><li><code><em>&lt;read_timeout&gt;</em></code>: Enter the number of seconds to wait to read from the back-end app, for example <strong>65s</strong>.</li></ul></td>
  </tr>
  </tbody></table>

  </dd></dl>

##### **Setting the maximum allowed size of the client request body**
{: #client_max_body_size}

Use this annotation to adjust the size of the body that the client can send as part of a request.
{:shortdesc}

<dl>
<dt>Description</dt>
<dd>To maintain the expected performance, the maximum client request body size is set to 1 megabyte. When a client request with a body size over the limit is sent to the Ingress controller, and the client does not allow to split up data into multiple chunks, the Ingress controller returns a 413 (Request Entity Too Large) http response to the client. A connection between the client and the Ingress controller is not possible until the size of the request body is reduced. When the client allows data to be split up into multiple chunks, data is divided into packages of 1 megabyte and sent to the Ingress controller.

</br></br>
You might want to increase the maximum body size because you expect client requests with a body size that is greater than 1 megabyte. For example, you want your client to be able to upload big files. Increasing the maximum request body size might impact the performance of your Ingress controller because the connection to the client must stay open until the request is received.
</br></br>
<strong>Note:</strong> Some client web browsers cannot display the 413 http response message properly.</dd>
<dt>Sample Ingress resource YAML</dt>
<dd>

<pre class="codeblock">
<code>apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myingress
  annotations:
    ingress.bluemix.net/client-max-body-size: "&lt;size&gt;"
spec:
  tls:
  - hosts:
    - mydomain
    secretName: mytlssecret
  rules:
  - host: mydomain
    http:
      paths:
      - path: /
        backend:
          serviceName: myservice
          servicePort: 8080</code></pre>

<table>
  <thead>
  <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
  </thead>
  <tbody>
  <tr>
  <td><code>annotations</code></td>
  <td>Replace the following value:<ul><li><code><em>&lt;size&gt;</em></code>: Enter the maximum size of the client response body. For example, to set it to 200 megabyte, define <strong>200m</strong>.

  </br></br>
  <strong>Note:</strong> You can set the size to 0 to disable the check of the client request body size.</li></ul></td>
  </tr>
  </tbody></table>

  </dd></dl>


## Managing IP addresses and subnets
{: #cs_cluster_ip_subnet}

You can use portable public subnets and IP addresses to expose apps in your cluster and make them accessible from the internet.
{:shortdesc}

In {{site.data.keyword.containershort_notm}}, you can add stable, portable IPs for Kubernetes services by adding network subnets to the cluster. When you create a standard cluster, {{site.data.keyword.containershort_notm}} automatically provisions a portable public subnet and 5 IP addresses. Portable public IP addresses are static and do not change when a worker node, or even the cluster, is removed.

One of the portable public IP addresses is used for the [Ingress controller](#cs_apps_public_ingress) that you can use to expose multiple apps in your cluster by using a public route. The remaining 4 portable public IP addresses can be used to expose single apps to the public by [creating a load balancer service](#cs_apps_public_load_balancer).

**Note:** Portable public IP addresses are charged on a monthly basis. If you choose to remove portable public IP addresses after your cluster is provisioned, you still have to pay the monthly charge, even if you used them only for a short amount of time.



1.  Create a Kubernetes service configuration script that is named `myservice.yaml` and define a service of type `LoadBalancer` with a dummy load balancer IP address. The following example uses the IP address 1.1.1.1 as the load balancer IP address.

    ```
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        run: myservice
      name: myservice
      namespace: default
    spec:
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        run: myservice
      sessionAffinity: None
      type: LoadBalancer
      loadBalancerIP: 1.1.1.1
    ```
    {: codeblock}

2.  Create the service in your cluster.

    ```
    kubectl apply -f myservice.yaml
    ```
    {: pre}

3.  Inspect the service.

    ```
    kubectl describe service myservice
    ```
    {: pre}

    **Note:** The creation of this service fails because the Kubernetes master cannot find the specified load balancer IP address in the Kubernetes config map. When you run this command, you can see the error message and the list of available public IP addresses for the cluster.

    ```
    Error on cloud load balancer a8bfa26552e8511e7bee4324285f6a4a for service default/myservice with UID 8bfa2655-2e85-11e7-bee4-324285f6a4af: Requested cloud provider IP 1.1.1.1 is not available. The following cloud provider IPs are available: <list_of_IP_addresses>
    ```
    {: screen}

</staging>

### Freeing up used public IP addresses
{: #freeup_ip}

You can free up a used portable public IP address by deleting the load balancer service that is using the portable public IP address.

[Before you begin, set the context for the cluster you want to use.](cs_cli_install.html#cs_cli_configure)

1.  List available services in your cluster.

    ```
    kubectl get services
    ```
    {: pre}

2.  Remove the load balancer service that uses a public IP address.

    ```
    kubectl delete service <myservice>
    ```
    {: pre}


## Deploying apps with the GUI
{: #cs_apps_ui}

When you deploy an app to your cluster by using the Kubernetes dashboard, a deployment is automatically created for you that creates, updates, and manages the pods in your cluster.
{:shortdesc}

Before you begin:

-   Install the required [CLIs](cs_cli_install.html#cs_cli_install).
-   [Target your CLI](cs_cli_install.html#cs_cli_configure) to your cluster.

To deploy your app:

1.  [Open the Kubernetes dashboard](#cs_cli_dashboard).
2.  From the Kubernetes dashboard, click **+ Create**.
3.  Select **Specify app details below** to enter the app details on the GUI or **Upload a YAML or JSON file** to upload your app [configuration file ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/). Use [this example YAML file ![External link icon](../icons/launch-glyph.svg "External link icon")](https://github.com/IBM-{{site.data.keyword.Bluemix_notm}}/kube-samples/blob/master/deploy-apps-clusters/deploy-ibmliberty.yaml) to deploy a container from the **ibmliberty** image in the US-South region.
4.  In the Kubernetes dashboard, click **Deployments** to verify that the deployment was created.
5.  If you made your app publicly available by using a node port service, a load balancer service, or Ingress, [verify that you can access the app](#cs_apps_public).

## Deploying apps with the CLI
{: #cs_apps_cli}

After a cluster is created, you can deploy an app into that cluster by using the Kubernetes CLI.
{:shortdesc}

Before you begin:

-   Install the required [CLIs](cs_cli_install.html#cs_cli_install).
-   [Target your CLI](cs_cli_install.html#cs_cli_configure) to your cluster.

To deploy your app:

1.  Create a configuration script based on [Kubernetes best practices ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/concepts/configuration/overview/). Generally, a configuration script contains configuration details for each of the resources you are creating in Kubernetes. Your script might include one or more of the following sections:

    -   [Deployment ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/): Defines the creation of pods and replica sets. A pod includes an individual containerized app and replica sets control multiple instances of pods.

    -   [Service ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/concepts/services-networking/service/): Provides front-end access to pods by using a worker node or load balancer public IP address, or a public Ingress route.

    -   [Ingress ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/concepts/services-networking/ingress/): Specifies a type of load balancer that provides routes to access your app publicly.

2.  Run the configuration script in a cluster's context.

    ```
    kubectl apply -f deployment_script_location
    ```
    {: pre}

3.  If you made your app publicly available by using a node port service, a load balancer service, or Ingress, [verify that you can access the app](#cs_apps_public).



## Managing rolling deployments
{: #cs_apps_rolling}

You can manage the rollout of your changes in an automated and controlled fashion. If your rollout isn't going according to plan, you can roll back your deployment to the previous revision.
{:shortdesc}

Before you begin, create a [deployment](#cs_apps_cli).

1.  [Roll out ![External link icon](../icons/launch-glyph.svg "External link icon")](https://kubernetes.io/docs/user-guide/kubectl/v1.5/#rollout) a change. For example, you might want to change the image that you used in your initial deployment.

    1.  Get the deployment name.

        ```
        kubectl get deployments
        ```
        {: pre}

    2.  Get the pod name.

        ```
        kubectl get pods
        ```
        {: pre}

    3.  Get the name of the container that is running in the pod.

        ```
        kubectl describe pod <pod_name>
        ```
        {: pre}

    4.  Set the new image for the deployment to use.

        ```
        kubectl set image deployment/<deployment_name><container_name>=<image_name>
        ```
        {: pre}

    When you run the commands, the change is immediately applied and logged in the roll-out history.

2.  Check the status of your deployment.

    ```
    kubectl rollout status deployments/<deployment_name>
    ```
    {: pre}

3.  Roll back a change.
    1.  View the roll-out history for the deployment and identify the revision number of your last deployment.

        ```
        kubectl rollout history deployment/<deployment_name>
        ```
        {: pre}

        **Tip:** To see the details for a specific revision, include the revision number.

        ```
        kubectl rollout history deployment/<deployment_name> --revision=<number>
        ```
        {: pre}

    2.  Roll back to the previous version, or specify a revision. To roll back to the previous version, use the following command.

        ```
        kubectl rollout undo deployment/<depoyment_name> --to-revision=<number>
        ```
        {: pre}

## Adding {{site.data.keyword.Bluemix_notm}} services
{: #cs_apps_service}

Encrypted Kubernetes secrets are used to store {{site.data.keyword.Bluemix_notm}} service details and credentials and allow secure communication between the service and the cluster. As the cluster user, you can access this secret by mounting it as a volume to a pod.
{:shortdesc}

Before you begin, [target your CLI](cs_cli_install.html#cs_cli_configure) to your cluster. Make sure that the {{site.data.keyword.Bluemix_notm}} service that you want to use in your app was [added to the cluster](cs_cluster.html#cs_cluster_service) by the cluster admin.

Kubernetes secrets are a secure way to store confidential information, such as user names, passwords, or keys. Rather than exposing confidential information via environment variables or directly in the Dockerfile, secrets must be mounted as a secret volume to a pod in order to be accessible by a running container in a pod.

When you mount a secret volume to your pod, a file named binding is stored in the volume mount directory that includes all information and credentials that you need to access the {{site.data.keyword.Bluemix_notm}} service.

1.  List available secrets in your cluster namespace.

    ```
    kubectl get secrets --namespace=<my_namespace>
    ```
    {: pre}

    Example output:

    ```
    NAME                                    TYPE                                  DATA      AGE
    binding-<service_instance_name>         Opaque                                1         3m

    ```
    {: screen}

2.  Look for a secret of type **Opaque** and note the **name** of the secret. If multiple secrets exist, contact your cluster administrator to identify the correct service secret.

3.  Open your preferred editor.

4.  Create a YAML file to configure a pod that can access the service details through a secret volume.

    ```
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      labels:
        app: secret-test
      name: secret-test
      namespace: <my_namespace>
    spec:
      selector:
        matchLabels:
          app: secret-test
      replicas: 1
      template:
        metadata:
          labels:
            app: secret-test
        spec:
          containers:
          - image: nginx
            name: secret-test
            volumeMounts:
            - mountPath: /opt/service-bind
              name: service-bind-volume
          volumes:
          - name: service-bind-volume
            secret:
              defaultMode: 420
              secretName: binding-<service_instance_name>
    ```
    {: codeblock}

    <table>
    <thead>
    <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
    </thead>
    <tbody>
    <tr>
    <td><code>volumeMounts/mountPath</code></td>
    <td>The name of the secret volume that you want to mount to your container.</td>
    </tr>
    <tr>
    <td><code>volumes/name</code></td>
    <td>Enter a name for the secret volume that you want to mount to your container.</td>
    </tr>
    <tr>
    <td><code>secret/defaultMode</code></td>
    <td>Set read-only permissions for the service secret.</td>
    </tr>
    <tr>
    <td><code>secret/secretName</code></td>
    <td>Enter the name of the secret that you noted earlier.</td>
    </tr></tbody></table>

5.  Create the pod and mount the secret volume.

    ```
    kubectl apply -f <yaml_path>
    ```
    {: pre}

6.  Verify that the pod is created.

    ```
    kubectl get pods --namespace=<my_namespace>
    ```
    {: pre}

    Example CLI output:

    ```
    NAME                           READY     STATUS    RESTARTS   AGE
    secret-test-1111454598-gfx32   1/1       Running   0          1m
    ```
    {: screen}

7.  Note the **NAME** of your pod.
8.  Get the details about the pod and look for the secret name.

    ```
    kubectl describe pod <pod_name>
    ```
    {: pre}

    Output:

    ```
    ...
    Volumes:
      service-bind-volume:
        Type:       Secret (a volume populated by a Secret)
        SecretName: binding-<service_instance_name>
    ...
    ```
    {: screen}

    

9.  When implementing your app, configure it to find the secret file named **binding** in the mount directory, parse the JSON content and determine the URL and service credentials to access your {{site.data.keyword.Bluemix_notm}} service.

You can now access the {{site.data.keyword.Bluemix_notm}} service details and credentials. To work with your {{site.data.keyword.Bluemix_notm}} service, make sure your app is configured to find the service secret file in the mount directory, parse the JSON content and determine the service details.

## Creating persistent storage
{: #cs_apps_volume_claim}

You create a persistent volume claim to provision NFS file storage for your cluster. You mount this claim to a pod to ensure that data is available even if the pod crashes or shuts down.
{:shortdesc}

The NFS file storage that backs the persistent volume is clustered by IBM in order to provide high availability for your data.

1.  Review the available storage classes. {{site.data.keyword.containerlong}} provides three pre-defined storage classes so that the cluster admin does not have to create any storage classes.

    ```
    kubectl get storageclasses
    ```
    {: pre}

    ```
    $ kubectl get storageclasses
    NAME                         TYPE
    ibmc-file-bronze (default)   ibm.io/ibmc-file
    ibmc-file-gold               ibm.io/ibmc-file
    ibmc-file-silver             ibm.io/ibmc-file
    ```
    {: screen}

2.  Review the IOPS of a storage class or the available sizes.

    ```
    kubectl describe storageclasses ibmc-file-silver
    ```
    {: pre}

    The **parameters** field provides the IOPS per GB associated with the storage class and the available sizes in gigabytes.

    ```
    Parameters: iopsPerGB=4,sizeRange=20Gi,40Gi,80Gi,100Gi,250Gi,500Gi,1000Gi,2000Gi,4000Gi,8000Gi,12000Gi
    ```
    {: screen}

3.  In your preferred text editor, create a configuration script to define your persistent volume claim and save the configuration as a `.yaml` file.

    ```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: <pvc_name>
      annotations:
        volume.beta.kubernetes.io/storage-class: "ibmc-file-silver"
    spec:
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 20Gi
    ```
    {: codeblock}

    <table>
    <thead>
    <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
    </thead>
    <tbody>
    <tr>
    <td><code>metadata/name</code></td>
    <td>Enter the name of the persistent volume claim.</td>
    </tr>
    <tr>
    <td><code>metadata/annotations</code></td>
    <td>Specify the storage class that defines the IOPS per GB of the host file share for the persistent volume:<ul><li>ibmc-file-bronze: 2 IOPS per GB.</li><li>ibmc-file-silver: 4 IOPS per GB.</li><li>ibmc-file-gold: 10 IOPS per GB.</li>

    </li> If you do no specify a storage class, the persistent volume is created with the bronze storage class.</td>
    </tr>
    <tr>
    <td><code>spec/accessModes</code>
    <code>resources/requests/storage</code></td>
    <td>If you choose a size other than one that is listed, the size is rounded up. If you select a size larger than the largest size, then the size is rounded down.</td>
    </tr>
    </tbody></table>

4.  Create the persistent volume claim.

    ```
    kubectl apply -f <local_file_path>
    ```
    {: pre}

5.  Verify that your persistent volume claim is created and bound to the persistent volume. This process can take a few minutes.

    ```
    kubectl describe pvc <pvc_name>
    ```
    {: pre}

    Your output looks similar to the following.

    ```
    Name:  <pvc_name>
    Namespace: default
    StorageClass: ""
    Status:  Bound
    Volume:  pvc-0d787071-3a67-11e7-aafc-eef80dd2dea2
    Labels:  <none>
    Capacity: 20Gi
    Access Modes: RWX
    Events:
      FirstSeen LastSeen Count From        SubObjectPath Type  Reason   Message
      --------- -------- ----- ----        ------------- -------- ------   -------
      3m  3m  1 {ibm.io/ibmc-file 31898035-3011-11e7-a6a4-7a08779efd33 }   Normal  Provisioning  External provisioner is provisioning volume for claim "default/my-persistent-volume-claim"
      3m  1m  10 {persistentvolume-controller }       Normal  ExternalProvisioning cannot find provisioner "ibm.io/ibmc-file", expecting that a volume for the claim is provisioned either manually or via external software
      1m  1m  1 {ibm.io/ibmc-file 31898035-3011-11e7-a6a4-7a08779efd33 }   Normal  ProvisioningSucceeded Successfully provisioned volume pvc-0d787071-3a67-11e7-aafc-eef80dd2dea2

    ```
    {: screen}

6.  {: #cs_apps_volume_mount}To mount the persistent volume claim to your pod, create a configuration script. Save the configuration as a `.yaml` file.

    ```
    apiVersion: v1
    kind: Pod
    metadata:
     name: <pod_name>
    spec:
     containers:
     - image: nginx
       name: mycontainer
       volumeMounts:
       - mountPath: /volumemount
         name: myvol
     volumes:
     - name: myvol
       persistentVolumeClaim:
         claimName: mypvc
    ```
    {: codeblock}

    <table>
    <thead>
    <th colspan=2><img src="images/idea.png"/> Understanding the YAML file components</th>
    </thead>
    <tbody>
    <tr>
    <td><code>metadata/name</code></td>
    <td>The name of the pod.</td>
    </tr>
    <tr>
    <td><code>volumeMounts/mountPath</code></td>
    <td>The absolute path of the directory to where the volume is mounted inside the container.</td>
    </tr>
    <tr>
    <td><code>volumeMounts/name</code></td>
    <td>The name of the volume that you mount to your container.</td>
    </tr>
    <tr>
    <td><code>volumes/name</code></td>
    <td>The name of the volume that you mount to your container. Typically this name is the same as <code>volumeMounts/name</code>.</td>
    </tr>
    <tr>
    <td><code>volumes/name/persistentVolumeClaim</code></td>
    <td>The name of the persistent volume claim that you want to use as your volume. When you mount the volume to the pod, Kubernetes identifies the persistent volume that is bound to the persistent volume claim and enables the user to read from and write to the persistent volume.</td>
    </tr>
    </tbody></table>

7.  Create the pod and mount the persistent volume claim to your pod.

    ```
    kubectl apply -f <local_yaml_path>
    ```
    {: pre}

8.  Verify that the volume is successfully mounted to your pod.

    ```
    kubectl describe pod <pod_name>
    ```
    {: pre}

    The mount point is in the **Volume Mounts** field and the volume is in the **Volumes** field.

    ```
     Volume Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-tqp61 (ro)
          /volumemount from myvol (rw)
    ...
    Volumes:
      myvol:
        Type: PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
        ClaimName: mypvc
        ReadOnly: false

    ```
    {: screen}

## Adding non-root user access to persistent storage
{: #cs_apps_volumes_nonroot}

Non-root users do not have write permission on the volume mount path for NFS-backed storage. To grant write permission, you must edit the Dockerfile of the image to create a directory on the mount path with the correct permission.
{:shortdesc}

Before you begin, [target your CLI](cs_cli_install.html#cs_cli_configure) to your cluster.

If you are designing an app with a non-root user that requires write permission to the volume, you must add the following processes to your Dockerfile and entrypoint script:

-   Create a non-root user.
-   Temporarily add the user to the root group.
-   Create a directory in the volume mount path with the correct user permissions.

For {{site.data.keyword.containershort_notm}}, the default owner of the volume mount path is the owner `nobody`. With NFS storage, if the owner does not exist locally in the pod, then the `nobody` user is created. The volumes are set up to recognize the root user in the container, which for some apps, is the only user inside a container. However, many apps specify a non-root user other than `nobody` that writes to the container mount path.

1.  Create a Dockerfile in a local directory. This example Dockerfile is creating a non-root user named `myguest`.

    ```
    FROM registry.<region>.bluemix.net/ibmnode:latest

    # Create group and user with GID & UID 1010.
    # In this case your are creating a group and user named myguest.
    # The GUID and UID 1010 is unlikely to create a conflict with any existing user GUIDs or UIDs in the image.
    # The GUID and UID must be between 0 and 65536. Otherwise, container creation fails.
    RUN groupadd --gid 1010 myguest
    RUN useradd --uid 1010 --gid 1010 -m --shell /bin/bash myguest

    ENV MY_USER=myguest

    COPY entrypoint.sh /sbin/entrypoint.sh
    RUN chmod 755 /sbin/entrypoint.sh

    EXPOSE 22
    ENTRYPOINT ["/sbin/entrypoint.sh"]
    ```
    {: codeblock}

2.  Create the entrypoint script in the same local folder as the Dockerfile. This example entrypoint script is specifying `/mnt/myvol` as the volume mount path.

    ```
    #!/bin/bash
    set -e

    # This is the mount point for the shared volume.
    # By default the mount point is owned by the root user.
    MOUNTPATH="/mnt/myvol"
    MY_USER=${MY_USER:-"myguest"}

    # This function creates a subdirectory that is owned by
    # the non-root user under the shared volume mount path.
    create_data_dir() {
      #Add the non-root user to primary group of root user.
      usermod -aG root $MY_USER

      # Provide read-write-execute permission to the group for the shared volume mount path.
      chmod 775 $MOUNTPATH

      # Create a directory under the shared path owned by non-root user myguest.
      su -c "mkdir -p ${MOUNTPATH}/mydata" -l $MY_USER
      su -c "chmod 700 ${MOUNTPATH}/mydata" -l $MY_USER
      ls -al ${MOUNTPATH}

      # For security, remove the non-root user from root user group.
      deluser $MY_USER root

      # Change the shared volume mount path back to its original read-write-execute permission.
      chmod 755 $MOUNTPATH
      echo "Created Data directory..."
    }

    create_data_dir

    # This command creates a long-running process for the purpose of this example.
    tail -F /dev/null
    ```
    {: codeblock}

3.  Log in to {{site.data.keyword.registryshort_notm}}.

    ```
    bx cr login
    ```
    {: pre}

4.  Build the image locally. Remember to replace _&lt;my_namespace&gt;_ with the namespace to your private images registry. Run `bx cr namespace-get` if you need to find your namespace.

    ```
    docker build -t registry.<region>.bluemix.net/<my_namespace>/nonroot .
    ```
    {: pre}

5.  Push the image to your namespace in {{site.data.keyword.registryshort_notm}}.

    ```
    docker push registry.<region>.bluemix.net/<my_namespace>/nonroot
    ```
    {: pre}

6.  Create a persistent volume claim by creating a configuration `.yaml` file. This example uses a lower performance storage class. Run `kubectl get storageclasses` to see available storage classes.

    ```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mypvc
      annotations:
        volume.beta.kubernetes.io/storage-class: "ibmc-file-bronze"
    spec:
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 20Gi
    ```
    {: codeblock}

7.  Create the persistent volume claim.

    ```
    kubectl apply -f <local_file_path>
    ```
    {: pre}

8.  Create a configuration script to mount the volume and run the pod from the nonroot image. The volume mount path `/mnt/myvol` matches the mount path that is specified in the Dockerfile. Save the configuration as a `.yaml` file.

    ```
    apiVersion: v1
    kind: Pod
    metadata:
     name: mypod
    spec:
     containers:
     - image: registry.<region>.bluemix.net/<my_namespace>/nonroot
       name: mycontainer
       volumeMounts:
       - mountPath: /mnt/myvol
         name: myvol
     volumes:
     - name: myvol
       persistentVolumeClaim:
         claimName: mypvc
    ```
    {: codeblock}

9.  Create the pod and mount the persistent volume claim to your pod.

    ```
    kubectl apply -f <local_yaml_path>
    ```
    {: pre}

10. Verify that the volume is successfully mounted to your pod.

    ```
    kubectl describe pod mypod
    ```
    {: pre}

    The mount point is listed in the **Volume Mounts** field and the volume is listed in the **Volumes** field.

    ```
     Volume Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-tqp61 (ro)
          /mnt/myvol from myvol (rw)
    ...
    Volumes:
      myvol:
        Type: PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
        ClaimName: mypvc
        ReadOnly: false

    ```
    {: screen}

11. Log in to the pod after the pod is running.

    ```
    kubectl exec -it mypod /bin/bash
    ```
    {: pre}

12. View permissions of your volume mount path.

    ```
    ls -al /mnt/myvol/
    ```
    {: pre}

    ```
    root@instance-006ff76b:/# ls -al /mnt/myvol/
    total 12
    drwxr-xr-x 3 root    root    4096 Jul 13 19:03 .
    drwxr-xr-x 3 root    root    4096 Jul 13 19:03 ..
    drwx------ 2 myguest myguest 4096 Jul 13 19:03 mydata
    ```
    {: screen}

    This output shows that root has read, write, and execute permissions on the volume mount path `mnt/myvol/`, but the non-root myguest user has permission to read and write to the `mnt/myvol/mydata` folder. Because of these updated permissions, the non-root user can now write data to the persistent volume.


