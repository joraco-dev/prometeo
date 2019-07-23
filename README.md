# Prometeo


In front of natural disasters, such as Wildfires, PROMETEO tries to protect those who protect us, the firefighters.

With our solution, we've developed a prototype sensor which sends basic telemetry (temperature, humidity and smoke concentration), these data is processed by a machine learning algorithm which will be able to predict if the health of the firefighter is ok, will be in danger or if it's in danger.

So, let us explain how we achieved this through this readme.

[Project website](https://github.com/joraco-dev/prometeo)

## The solution in a glance

![alt text](https://github.com/joraco-dev/prometeo/blob/master/content/Presentation.png)

As you can see, our project has 5 major blocks, The Sensor, IBM IoT Hub, NodeRed, Container service and the client. Lets review them one by one.

### The Sensor

For this part of the project, we used a board based on the ESP8266 micro-controller with NodeMCU firmware, as this project will be open source. To this controller, we attached a couple of sensors, the DHT11 which give us information about Temperature and Humidity and the MQ-2 sensor which give us, among the concentration of several gases, the concentration of smoke.

In order to do it portable, we decided to power the controller with a portable usb battery and wrap-it with a sports arm band.

The following image is the final result with a firefighter wearing it.

![alt text](https://raw.githubusercontent.com/imagen de joan y vicenç con el sensor puesto)

Finally, the code that makes possible to read those metrics and send it to the next stage (IBM IoT Hub) could be reviewed here. There are some things to have in mind reading this code:
	
- The connectivity to the internet is intended to do it with tethering with a mobile phone, so you need to supply SSID and password of the wifi-hotspot.
- You need to take into consideration that the pins will vary if you decide to us another board or type of sensors, in our case for the Temperature and Humidity we use de the digital pin 5 and for the smoke sensor we used the analog pin 0.
- You will need a valid token for the IoT Hub and register your device in the Hub
- We based our code on the example provided on the IBM Developer portal written by Ant Elder (https://developer.ibm.com/recipes/tutorials/run-an-esp8266arduino-as-a-iot-foundation-managed-device/)
	

### IBM IoT HUB

This was very straight forward step, simply register the new devices and make sure they connect to the platform, also and thanks to the coded uploaded to pur microcontroller we're able to perform remote actions on our decide, such as changing the polling interval, restart the device and wipe it.

Also, and it was a very important step, we create a connection between the IoT Hub and our next step, our NodeRed app.

![alt text](https://raw.githubusercontent.com/imagen de la consola)

### NodeRed

![alt text](https://raw.githubusercontent.com/imagen de nodered)

At this point, we can talk that we're in front of our service core. With this app we control all the workflow of the metrics sent by our sensors, store them, analyze them and take actions depending on the readings.

So, lets analyse node by node. Also, you can find the code here, if you want to import to your personal project, just take into account that credentials, tokens and sensitive data are deleted.

- IBM IoT: It connects and receives the events from every device registered in our IoT Hub. The messages are received in json format.
	
	Once the message is received we take two actions in parallel
- nodemcu: Tith this node, we save a cpy of the message in a cloudant database, wit this we will have historical data for the future.
- IoT2ML: At this function node, we only transform the message received in order to make it comprehensive by our machine learning service.
	
- Machine Learning Firefighter Health Prediction: in the watson machine learning is where the "magic" happens, thanks to our predictive model, once we send the metrics, our model will reply with a Green, yellow or red firefighter status. We will go deeper on our explanation on the Watson Machine Learning section below.

- ML2status2.0: At this point, we finally prepare the message in order to be sent to our live dashboard. Basically we send the follwoing paylod, "Firefighter ID", "Status", "Timestamp of the event", "Temperature", "Humidity", "Smoke concentration"

- Webscokets Server: This is the end node, which sends the messages to our websockets send and receive server, later we will talk more in detail.

## Watson Machine Learning

### IBM Cloud Container Service

At this point, we need somewhere to publish our real time dashboard. We created a service at the IBM Cloud Container Service, this service includes a websockets receiver and sender (you can see the code here) and a NGINX serving our portal wirtten basically with javascript and using datatables library based on jquery (also, the code of the web page could be reviewd here)

This service has exposed two ports, one for the websockets server and the other for the nginx server.

The following script is the one we use in order to deploy and updated version of our POD.

```
#!/bin/sh
export KUBECONFIG=/root/kubeconf_ibm/kube-config-mil01-mycluster.yml
service='wsserv-deployment'
docker build --tag uk.icr.io/nodemcu/wsserv:1 .
docker push uk.icr.io/nodemcu/wsserv:1
kubectl delete "pod/`kubectl get pods --no-headers=true|awk '{print $1}'`"

ports=`kubectl get service/${service} --no-headers=true|awk '{print $5}'`
port_ws=`kubectl get service/${service} --no-headers=true|awk '{print $5}'|sed 's/:/ /g'|sed 's/\/TCP//g'|sed 's/,/ /g'|awk '{print $2}'`
port_http=`kubectl get service/${service} --no-headers=true|awk '{print $5}'|sed 's/:/ /g'|sed 's/\/TCP//g'|sed 's/,/ /g'|awk '{print $4}'`
ip=`kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'`

echo "http://"$ip":"$port_http"\n"
echo "ws://"$ip":"$port_ws"\n"
```


Explain how to run the automated tests for this system

### Break down into end to end tests

Explain what these tests test and why

```
Give an example
```

### And coding style tests

Explain what these tests test and why

```
Give an example
```

## Deployment

Add additional notes about how to deploy this on a live system

## Built With

* [Dropwizard](http://www.dropwizard.io/1.0.2/docs/) - The web framework used
* [Maven](https://maven.apache.org/) - Dependency Management
* [ROME](https://rometools.github.io/rome/) - Used to generate RSS Feeds

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Billie Thompson** - *Initial work* - [PurpleBooth](https://github.com/PurpleBooth)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the Apache 2 License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Based on [Billie Thompson's README template](https://gist.github.com/PurpleBooth/109311bb0361f32d87a2).
