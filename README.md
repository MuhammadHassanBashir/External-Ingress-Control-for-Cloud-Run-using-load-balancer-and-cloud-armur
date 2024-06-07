# External-Ingress-Control-for-Cloud-Run-using-load-balancer-and-cloud-armur
We can whitelist ip address with cloud aumur to restricting cloudrun access only for the specfic ips.

     Cloud Run is a fully managed service provided by Google Cloud that allows you to deploy containerised applications. In deploying a container image to Cloud Run, you create what’s called a service. These services have an immutable URL associated with them which can be used to access your app. While you can lock them down using IAM, there’s always additional security benefits to using external ingress controls and whitelisting IPs from trusted sources. After all, you don’t want just anybody accessing your app.

**How do we do it?** 

    Cloud Load Balancing and Cloud Armor! Network-based ingress controls can be implemented with the help of a load balancer that has security policies attached to it, ensuring that only trusted sources can access the load balancer. This can be done in addition to allowing only authenticated invocations of the service. For this tutorial, we’ll be whitelisting our own IP address.
    
**We’ll be going through how we can set this up, step-by-step.** 

    Begin by following this Quickstart tutorial provided by Google Cloud to set up our Cloud Run service (pick any language you’re comfortable with). Once you’ve got your container up and running, register a domain name for your service (we will be using this later).  

**Cloud Load Balancing:**

    Cloud Load Balancing is a service that helps you manage traffic to your resources. We’re going to set up an HTTPS Load Balancer for our service.

    - Head over to the Load Balancing page in the console. Start the configuration for HTTP(S) Load Balancing.
    - Choose “From internet to my VMs” for the Internet facing or internal only option, and click continue. Choose an appropriate name for your load balancer.
    - Now you’ll have to configure the load balancer’s host rules and path matchers, backend, and frontend.
    
    steps:
      - click on create load balancer
      - Select **Application Load Balancer(HTTP/HTTPS)** as load balancer type.
      - Select **Public facing(external)**
      - Select **Best for global workloads** in Global or single region deployment.
      - Select **Global external Application load balancer** in load balancer generation.
      - select create load balancer and then click on configure

**Frontend Configuration**
    
    for protocal HTTPS:
    
    - Click Frontend configuration, and enter a Name.
    - protocal: HTTPS(includes HTTP/2 and HTTP/3)
    - ip version: IPv4, ip address: Ephermeral, port: 443
    - certificate: click on create a new certificate , then click on create Google-managed certificate. , then give the domain name: tttest.disearch.ai, click on create (Create a free Google-managed SSL certificate for the domain name you registered earlier. This is needed to create an HTTPS Load Balancer.)
    - Click Done.

    for protocal HTTP:

    - Click Frontend configuration, and enter a Name.
    - protocal: HTTP
    - ip version: IPv4, ip address: Ephermeral, port: 80

**Backend Configuration**

    -  Click on create a BACKEND Service
    -  Select Backend type: Serverless network endpoint group(for selecting Cloud Run as backend)
    -  Protocol: HTTPS
    -  Select Backends>New backends>Create serverless network endpoint group.
    -  Choose a name for your serverless NEG.
    -  Pick the region your container is running in.
    -  Select Serverless network endpoint group type>Cloud Run. Select the cloud run **disearch** service you’ve created from the Select service name dropdown.
    -  Click Create.
  
    - Finally, click Done under New backend window and click Create.
  
    **Host Rules and Path Matchers**
  
    - Leave the host rules and path matchers to their default values, meaning that all requests go to the service you choose in the backend.
  
      We’ve configured all aspects of the load balancer, and can now click Create. When it is done being created, click on the name of the load balancer and note the IP address associated with it in the Frontend configuration. Point the DNS A record of your domain name to this IP address. This is the URL we will use to access our service.
  
**Domain Pointing:**
  
    - take ip address from frontend against 443 port and set domoin on gcp cloud dns section.

    Congratulations! We’re nearly done. All we need to do now is to add IP whitelisting.

**Cloud Armor**
    
    Cloud Armor is a Google offering that provides a web application firewall to control traffic to your applications. Find out what your IP address is, and head to the Cloud Armor page.
    
    Click Create policy and choose a Name for the policy.
    Under Default rule action, click Deny.
    Under Add more rules, click Add Rule.
    Under Mode, click Basic.
    Under Match, enter your IP address.
    Under Action, click Allow.
    Under Priority, enter 0. This gives maximum priority for this rule.
    Click Done.
    Under Apply policy to targets, click Add Targets.
    Under Type, select Load Balancer backend service
    Under Target, select the backend service created before
    Click Done.
    Finally, click Create policy. We’ve asked Cloud Armor to deny all traffic to the load balancer, with the exception of our IP address. You can also add a response for denied traffic(like 403 forbidden).
      
Now verify domain on browser and you will see that cloud run will be accessing through cloud armur whitelist-ips. (remember it takes some time for provisioning)
    
    
    
