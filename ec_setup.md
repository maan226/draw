# EC Setup- Attempt-1 
# USE CASE:  Connecting to Postgres DB on Predix
#### Step-1
1.Open cmd – as admin
  *Use VPN if outside GE n/w because of the http&https proxies added
    
2.Login to CF1 using the following command:
```cf login -a https://api.system.aws-usw02-dev.ice.predix.io -u prasad_alokam@hotmail.com -p eC7eamR0ck! -o enterprise-connect -s Dev```

3.Create a new folder and change location to the folder:
> ECSetup (preferably in Desktop)
``` 
    cd /pthOfECSetup
```
4.To open the marketplace. Optional step
``` cf m ```
5.Download/ clone & extract the items in the given github link into the ECSetup folder
https://github.com/Enterprise-connect/ec-agent-cf-push-sample.
and 
https://github.build.ge.com/Enterprise-Connect/ec-sdk/tree/beta

6.Create a predix uaa sevice (can be reused if already present)
```cf create-service predix-uaa Free maanasaa-demo-uaa -c  uaa_creation.json```
where the json file is in the same folder(ECSetup) and contains this:
> {"adminClientSecret":"demo"}

7.Bind the service to an existing app in the same Dev space to ensure validation
```
    cf bind-service pwof-021318-server maanasaa-demo-uaa
    cf env pwof-021318-server 
```
or if unable to bind, use the following commands:
``` 
    cf csk maanasaa-demo-uaa demo-uaa-key
    cf service-key maanasaa-demo-uaa demo-uaa-key
```
a json with the VCAP credentials of the service is returned, save it in a separate file "maanasaa-demo-uaa.json"

8.Save the IssuerId value from the json retured after the above step in a separate json "ec_id.json"
  >  {"trustedIssuerIds":["https://9a0f0daa-a889-45f5-bcf7-8d2a941724ee.predix-uaa.run.aws-usw02-dev.ice.predix.io/oauth/token"]} 
 
9.In case you want to repeat the steps, unbind the service from the app or delete the service key and delete the service before recreating the same service.The commands are as follows:
```
    cf unbind-service pwof-021318-server maanasaa-demo-uaa
    cf delete-service maanasaa-demo-uaa
```
or
``` 
    cf delete-service-key maanasaa-demo-uaa demo-uaa-key
    cf delete-service maanasaa-demo-uaa
```
### Creating EC Service
#### Step 2

1.Create an EC service with the json with the issuer Id
```
    cf cs enterprise-connect Oneway-TLS ec-maanasaa-selfDemo1 -c ec_id.json
```

2. Bind this EC service with the same app as the uaa service
```
     cf bind-service pwof-021318-server ec-maanasaa-selfDemo1
     cf env pwof-021318-server
```
3. Save the VCAP returned into a separate json "pwof-021318-server_boundWith_ ec-maanasaa-selfDemo1.json"


#### UAA Clients
1. Open the UAA dashboard from the VCAP "dashboard url" in the maanasaa-demo-uaa.json file with the adminClientSecret as the password.
2. Create two new clients for the uaa 
    Check to include client_credentials and refresh tokens
3.Copy the OAuth scope from the VCAP "pwof-021318-server_boundWith_ ec-maanasaa-selfDemo1.json" and paste it in the Authorization scope of the two UAA clients created. Save the settings

##### Note
Configuring a unique UAA client for the EC-Server and Client is not mandatory.You can also reuse an existing UAA Client provided the corresponding UAA service is same as the one used to create the EC-Service. 

One UAA client can be used for multiple EC agents of many different EC-Services. 


### Push gateway to predix envt
#### Step 3
1. Copy the extracted ecagent_linux_sys and ecagent_windows_sys from the "ec-beta" folder to "ec-agent-cf-push" folder
2. Move to the location
``` cd /pathToec-agent-cf-push ```
3. open 
> ec.sh
4. Uncomment Gateway mode command 
    - replace value after -zon flag with http-header-value from VCAP "pwof-021318-server_boundWith_ ec-maanasaa-selfDemo1.json"
    - replace value after -sst flag with service-uri [without the "/beta/index"] from VCAP
    - replace value after -tkn flag with admin-token value from VCAP
    SAVE
```
    ./ecagent_linux_sys -mod gateway -lpt ${PORT} -zon e406dfbf-b5aa-4f00-83ea-a4e36c8bf77b -sst https://e406dfbf-b5aa-4f00-83ea-a4e36c8bf77b.run.aws-usw02-dev.ice.predix.io -tkn YWRtaW46Y1ZUUzNWbGtCSlBoOXprN0Jqb2lnOU5vM203RURVU3l3R3BwaWdtSm5LUGhvZENrYWo= -dbg
```
5. Open manifest.yml file and change the appName to  ec-maanasaa-DemoGateway
6. run the follwing command to push gateway to Predix and start the Application
```
    cf push ec-maanasaa-DemoGateway
    cf a
```
Save the gateway url temporarily for the app you just pushed
#### Step 4
1.Check for an existing postgres service instance using:
``` cf s```
2.  Bind the app "pwof-021318-server" to an existing postgres2.0 service instance.
``` 
      cf bind-service pwof-021318-server ec-init-test-postgres2
     cf env pwof-021318-server
```
Update the contents in "pwof-021318-server_boundWith_ ec-maanasaa-selfDemo1.json" with the VCAP for postgres instance too.
### push server to predix envt
#### Steps 5-7
1.Open 
> ec.sh
2. comment the gateway script and uncomment the server script. Edit
    -  replace value after -aid flag with any one of the two EC_info_id  from VCAP of EC
    - replace value after -hst flag with the gateway url
                 ```   wss://<gatewayurl>/agent```
    -replace value after -rht flag with host_id from the VCAP of Postgres
    -replace value after -cid flag with name of one of the two UAA clients created
    -replace value after -csc flag with the ClientSecret value for this UAA client
    -replace value after -oa2 flag with oauth url provided in the overview of UAA Dashboard or the trustedIssuerIds value
    SAVE
```
    ./ecagent_linux_sys -mod server -aid yPsXm8 -cid ec_maanasaa_client -csc demo -dur 1200 -hst wss://ec-maanasaa-demogateway.run.aws-usw02-dev.ice.predix.io/agent -oa2 https://9a0f0daa-a889-45f5-bcf7-8d2a941724ee.predix-uaa.run.aws-usw02-dev.ice.predix.io/oauth/token -zon e406dfbf-b5aa-4f00-83ea-a4e36c8bf77b -sst https://e406dfbf-b5aa-4f00-83ea-a4e36c8bf77b.run.aws-usw02-dev.ice.predix.io -rht db-0f1c7a77-1ccf-4cb1-9817-0c0e39f0bd1d.c7uxaqxgfov3.us-west-2.rds.amazonaws.com -rpt 5432 -dbg -hca ${PORT}
```
5.Open manifest.yml 
    Change the appName to "ec-maanasaa-DemoServer"
    
#### Step 8
1. Push the server using
    ```cf push ec-maanasaa-DemoServer ```

###  Client Instantiation
#### Steps 9 & 10
1. Open ec.sh
2. Comment server script and uncomment client.
    keep "ecagent_linux_sys" as it is if the client is a linux system. Change it to "ecagent_windows_sys" if client is windows
    -replace value after -aid flag with the other unused ec_info_id from VCAP of EC
    -replace value after -hst flag with gateway url
        ```   wss://<gatewayurl>/agent```
    -replace value after -lpt flag with the port number(any number to which the client will listen)
    -replace value after -tid with the "aid" value of server app
    -replace value after -oa2 flag with oauth url provided in the overview of UAA Dashboard or the trustedIssuerIds value
    -replace value after -cid flag with the name of the other UAA client
    -replace value after -csc flag with the ClientSecret for this client.
    -add a -pxy flag and the HTTP proxy after the flag.
 ```
    ecagent_windows_sys -mod client -aid OT8fBy -tid yPsXm8 -cid maanasaa_client -csc demo -dur 2000 -hst wss://ec-maanasaa-demogateway.run.aws-usw02-dev.ice.predix.io/agent -oa2 https://9a0f0daa-a889-45f5-bcf7-8d2a941724ee.predix-uaa.run.aws-usw02-dev.ice.predix.io/oauth/token -lpt 8080 -dbg -pxy "http://cis-americas-pitc-cinciz.proxy.corporate.gtm.ge.com:80"
```
#### Step 11   
1.Copy the client script and run it directly in cmd

###  Connecting to pgAdmin
#### Step 12
1.Download postgres2
2.open pgAdmin4 and open servers
3.Create a new server for the session "EC_Setup"

#### Step 13
1.In the connection_setup window:
-hostname -->localhost (as the EC client is run from the system locally)
-port --> same as the -lpt value in the EC Client
-Maintenance db -->postgres
-Username ---> should be taken from the VCAP of Postgres
-Password ---> should be taken from the VCAP of Postgres
-ssl  --->prefferd/default

2.Save
3.Connect to the DB server.
4. Open the postgres DB
5. Open any table and view the first 100 rows ( just an example)


