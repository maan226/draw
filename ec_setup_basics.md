## What is EC?
EC or Enterprise conenct is essentially a tool that lets the user connect any two end points. The potential end points could be anything between two private cloud systems, a public and a private cloud system or even an on premise system and a cloud system. It uses the mechanism of Web Sockets to establish a secure channel between the endpoints to facilitate data transfer or data access.
## Use Cases for EC
The use cases for EC are very diverse. It depends very highly on the imagination and the programming capability of the user.

Any potential use case requires the following conditions to be true.
* EC server can be run in any location that has visiblity to the desired data source .
* The data source can be represented with a valid IP
* The EC Client can be run in a location where the LOCALHOST convention will allow the tool/app trying to reach the data source to commmunicate.
* Use cases can belong to either of the two categories  :
    1. Reach-Back: Calling/ Going to call services or resources hosted in a public cloud (in this case Predix/ Cloud Foundry) .
    2. Reach-IN: The exact opposite of Reach-Back
* The only typical constant is the EC-Gateway which resides on Predix. 
* The user creates EC service by supplying the Predix UAA information.


## EC Setup
The setup of EC as such is a simple activity.
A typical EC Setup involves the creation of an EC-Service, an EC agent configured to work in the Server, Client and Gateway modes and the instances of the end application.

![alt text][simple_diagram]

[simple_diagram]: "https://github.com/maan226/draw/blob/master/general_explanation.png" 

### Note
The bearer token is fetched, first thing, upon startup of the EC Server and Client agents. In reality it is the Service API that checks the validity of the token and not the Gateway itself. 
While the EC Client uses the Gateway to pass the token to the Service API, the EC Server sends the token directly to the Service API without going through the Gateway. 

### An Example Setup.

![alt text][setup_diagram]

[setup_diagram]: "https://github.com/maan226/draw/blob/master/EC-Setup_gen.png" 

The above figure explain the general setup procedure of [EC setup] in a sequential fashion.

[EC setup]: https://github.com/maan226/draw/blob/master/ec_setup.md

##### Note
* The EC Service, Gateway and the Load Balancer reside on CloudFoundry
* The EC Server resides with the resource being accessed
* The EC Cient resides with the application accessing the resource.
* The configuration steps are largely similar across operating systems barring a few indegenious commands and the agent binary specific to the client system.
* For the example used, the operating systems of the Gateway and the Server are constant (Linux) as both are running on Predix. The change is seen only in the case of the EC client.
