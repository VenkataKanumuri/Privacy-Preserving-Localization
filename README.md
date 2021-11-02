# Privacy-Preserving Localization and Privileged Sensor Fusion in a Camera Network

## DESCRIPTION
This project involves designing a small-scale camera network using Raspberry Pi nodes to track a moving object. Each node will process the camera data to detect and track the object. The aggregation of the nodes’ data will rely on two cryptographically secure methods to ensure the privacy of some or all of the input data. 

This project can be divided into 4 parts.

### _Local Processing_:
* **Object Detection** 
>It is done at each sensor, and should produce a measurement in a form suitable for the encrypted information filter. We capture measurements of the  object (Laser pointer) in two dimensions on the plane parallel to the camera, obtaining global (x,y) position by fixing the origin which is common to all the cameras.

### _Networking_:
* **Distributed implementation**
> The sensors will be physically distributed and need to communicate to each other and a central node. Since the Raspberry Pi's will be on the same network and capable of pinging one another. We use TCP/IP handshakes to send messages from one machine to another machine (socket communication)

* **Synchronisation implementation**
> Sensors can make measurements and send them to the aggregation node and wait for confirmation that the measurements have been processed before making the next one. The central node will wait to receive a measurement from every sensor before processing them and reporting to the sensors that they may send the next measurement. 

### _Central Processing_:
* **Encrypted Multisensor Information Filtering**
> The information filter offers the advantage that updating estimates with sensor observations can be expressed in terms of simple summations. 
A central node generates a publicKey and a privateKey that matches the publickey and sends it to the sensors to encrypt the data. The file “ Central node” has the keys generated using the paillier key generation method used from the `phe.paillier` library. An *aggregated cipher text* is stored as objects in the dictionary *resultmap*. Central node which has the `getDecryptedSummation` function takes the arguments *resultmap* and privateKey and decrypts the aggregated cipher text using `paillier decrypt` function. These decrypted values are stored in the dictionary *“final resultmap”*. As the central node runs the *information filter*, these aggregated decrypted measurement vectors are given to the update step of the information filter where the information vector and matrices constantly gets updated which in return gives us the state estimates and its respective error covariance. Initially, the filter can be given a state with a very bad estimate<sub>x</sub>0 = [0, 0, 0, 0] and high error covariance. 

### _Cryptography_:
* **Privacy-preserving**

>We initially encrypt the measurment vector with the help of a publicKey, The publicKey is generated in the central node implementing pailler key generation method used from the `phe.pailer` library. Since the publicKey is available to encrypt the data, we use the “encrypt” method from ***paillier*** library to encrypt the measurement vector and store the cipher texts as objects in the dictionary “summation map”. This dictionary summation map has the indexes of measurement vectors. Now that we have all the encrypted values stored in the dictionary “summation map”, we have to aggregate the measurement vectors homomorphically using a ***paillier homomorphic addition*** operation.





## Environtment Setup:

The tabel below shows an overview of python lybraries we are using for these tasks.

| Distribution Name | Version   |       Description          |
| -------------     |:---------:| --------------------------:|
| Python            | 3.7       | Python is an interpreted, object-oriented, high-level programming language used in many software development practices. |
| Numpy             | 1.19      | Numpy is a python library that supports different operations to perform on multi dimensional arrays and matrices. |
| Opencv            | 4.5.2.54  | OpenCV-Python is a library of Python bindings designed to solve computer vision problems. |
| Matplotlib        | 3.3.1     | Matplotlib is a plotting library written in python framework. This library is helpful in creating static, animated and interactive visualizations in python. |
| python-socketio   | 5.4.0     | Socket.IO is a transport protocol that enables real-time bidirectional event-based communication between clients (typically, though not always, web browsers) and a server. |
| eventlet          | 0.32.0    | Eventlet is a concurrent networking library for Python that allows you to change how you run your code, not how you write it. |
| Phe               | 1.4.0     | A Python 3 library for Partially Homomorphic Encryption and Decryption.|


## Manual Install/Setup:

This section discusses software setup only, assuming you have hardware setup, the Rasberry Pi's are correctly connected to a common network, and also adjusted to detect the same object concurrently (localisation). (**edit about setup reference)

The below install is for manual operation of the required libraries in Raspberry Pi OS.

* Install, using apt-get, the following items: libatlas-base-dev

* Install remaining requirements using pip3 install for libraries mentioned in [requirements.txt](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/blob/main/requirements.txt)

To install all the libraries at once, run the [prerequisites.py](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/blob/main/prerequisites.py)


## Design and Implementation:

We have a total of 5 Rasberry Pi's which are being used as follows:
-  3 individual sensors
-  1 sensor as well as localHub (where local processing is done)
-  1 as Central node 

1. Individual Sensors:
> We deploy [ObjectTracking](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/blob/main/FinalCode/SensorNodes/Node1/ObjectTracking.py), [Measurements](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/blob/main/FinalCode/SensorNodes/Node1/Measurements.py), and [Encryption](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/blob/main/FinalCode/SensorNodes/Node1/Encryption.py) python files in each individual sensor. We track an object (red lazer light) projected within a common plane (localised for all the sensors) by using [ObjectTracking](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/blob/main/FinalCode/SensorNodes/Node1/ObjectTracking.py) script to get (x,y) coordinates. These coordinates are used in [Measurements](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/blob/main/FinalCode/SensorNodes/Node1/Measurements.py) to generate the measurements using a process following a linear constant velocity model. And these measurements are encrypted (Paillier encryption using PublicKey) by using [Encryption](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/blob/main/FinalCode/SensorNodes/Node1/Encryption.py) script. 

> In the above mentioned scripts except ObjectTracking.py, need to be deployed at each individual sensor nodes. For ObjectTracking.py, we need to calibrate the origin for each sensor seperatly in the below lines shown in the image.

<img  src="IMAGES/calibrationlines.png" alt="Calibration" width="600"/>


2. LocalHub Sensor:
> We deploy 3 python files deployed in each Individual sensors along with [LocalServer](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/LocalHubNode/LocalServer.py) and [SensorDataAggregation](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/LocalHubNode/SensorDataAggregation.py) python files in this LocalHub sensor. Here [SensorDataAggregation](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/LocalHubNode/SensorDataAggregation.py) is used to aggregate the cipher measurement vectors using Paillier homomorphic addition synchronously. This localised aggragated measurement is communicated to Central node for Decryption through [LocalServer](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/LocalHubNode/LocalServer.py)

3. Central Node:
> Here we generate Key (publicKey and PaillierPrivateKey) and we transport this publicKey (generated from [Decryption](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/CentralNode/Decryption.py)) using [CentralServer](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/CentralNode/CentralServer.py) to the sensors for Encryption. Also, the localised aggreagated measurement received from LocalHub sensor is decrypted at [Decryption](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/CentralNode/Decryption.py). This decrypted measurements (Localised sensor measurement) are used to run the [InformationFilter](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/CentralNode/InformationFilter.py).

![Communication map](IMAGES/PPLOC_Communication_map.png)


## Instructions to run the setup:
Note: Refer the above figure while going through the instructions
      Please do not move the position of the frame as the sensors are calibrated to a fixed origin.


1. Connect the monitor, keyboard+mouse to the **Central Node** (**Raspberry pi #3**). In the GUI of Raspberry Pi OS displayed on the monitor, Open task manager and navigate to the home directory (pi Home) and then to folder **ams-50_CentralServer** and run the [CentralServer.py](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/CentralNode/CentralServer.py) program (for reference please see below Central Node figure). The code will pop up on the **Thonny IDE**.

    [***Functionality:*** This Central server is run first to send the public key, created at Decryption program to each sensor to Encrypt their respective measurements]


2. Run the program by clicking on the play button and check whether the server has been started. It will be displayed in the IDE shell (shell will be displayed at the bottom of IDE).

    [***Functionality:*** The started central server displays the following: `wsgi starting up on http://<ipaddress>`]

<p align="center">
<img  width="800" height="500" src="IMAGES/CentralNode.jpeg" alt="Central Node"/>
</p>

3. Once the server has started, remove the monitor, keyboard+mouse and connect it to the **local hub** ( **Raspberry Pi #2**), open terminal and navigate to **ams-48_LocalServer+C** and run the [LocalServer.py](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/LocalHubNode/LocalServer.py) code (***Terminal window is preferred for this process as we use the Thonny IDE to run [ObjectTracking.py](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/LocalHubNode/ObjectTracking.py) code***).

    [***Functionality:*** The started local server displays the following:
    {PaillierPublicKey , 'N' value}
```
connection established
Server 
    wsgi starting up on http://<ipaddress>
``` 
]

4. After step 3, in the same **Pi #2** open task manager and navigate to home directory (pi Home) and then to folder **ams-48_LocalServer+C**, run the [ObjectTracking.py](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/LocalHubNode/ObjectTracking.py) code in the IDE.

    [***Functionality:*** The local server receives only one encrypted measurement when first sensor is run and waits for others to connect, then continuously prints polling requests (like `192.168.1.161 - - [03/Sep/2021 22:07:51] "POST /socket.io/?transport=polling&EIO=4&sid=Xn_GBOQhIzrW3wKXAAAA HTTP/1.1" 200 167 0.024792`) which we can ignore at this point

    At the ObectTracking.py end we get this output for the time being : `connection established ; Sending Data to server`]

<img  src="IMAGES/LocalHubNode.jpeg" alt="LocalHub Node" width="800"/>

5. Once the ***LocalServer*** is up and running in ***Pi #2***, as we also started one of the [ObjectTracking.py](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/LocalHubNode/ObjectTracking.py) program we can now tranfer the monitor and Keyboard+mouse to the next sensor (**Raspberry Pi #4**) and open task manager, navigate to home directory (pi Home), then to folder **ams-51_C** and run [ObjectTracking.py](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/SensorNodes/Node4/ObjectTracking.py) code in the IDE.

    [***Functionality:*** The local server receives another encrypted measurement and waits for remaining 2 to connect in the background.

    At this sensor we can see at ObectTracking.py end, this output for the time being : `connection established ; Sending Data to server`]

<img  src="IMAGES/SensorNode.jpeg" alt="Sensor Node" width="800"/>

6. Next, we will move to the sensor **Raspberry Pi #1** and again open task manager, and then we will navigate to home directory (pi Home), then to folder **ams-49_C** and run [ObjectTracking.py](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/SensorNodes/Node1/ObjectTracking.py) code in the IDE.

    [***Functionality:*** The local server receives another encrypted measurement and waits for remaining 1 to connect in the background.

    At this sensor we can see at ObectTracking.py end, this output for the time being : `connection established ; Sending Data to server`]

7. Now, repeat the above step at **Raspberry Pi #5**, by opening task manager, navigate to home directory (pi Home), then to folder **ams-52_C** and run [ObjectTracking.py](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/SensorNodes/Node5/ObjectTracking.py) code in the IDE.

    [***Functionality:*** Since the local server received all the encrypted measurements it displays 
    `Data received` from each node at ever time step (PublicKey for encryption)
    And 
    `phe.paillier.EncryptedNumber object ***********`

    From each sensor, local server receives Paillier objects and gets aggregated by homomorphic function and sends it to CentralNode for decryption
    in the background (can be verified by connecting to LocalServer and CentralServer again).

    At this sensor (and at other 3 sensors) we can see at *ObectTracking.py* end, outputs : 
    ```
    connection established
    Sending Data to server
    ACK received
    Sending Data to server
    ACK received
    ```
    ]   

8. We can now connect back to the **Central Node** (**Raspberry pi #3**) and review the IDE for results. There is also a graph plotted which will be saved as soon as all the nodes are up and running. The graph will be saved as ***plot.png*** in the `/home/ams-50_CentralServer directory`.

    [***Functionality:*** The Central node receives the encrypted data, decrypts it and runs the filter
    After which it sends acknowledgement to local server to send new set of data
    LocalServer intern sends acknowledgement to individual sensors to send next reading ]

- Note 1 :

>The plots at CentralServer home directory which are plotted in the file Plot.png will be overwritten for every new filter reading.
Incase we run the central server again we need to make sure we deleted the previous Plot.png and run the central server

- Note 2:
>The [LocalServer](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/LocalHubNode/LocalServer.py) waits for data from each sensor to establish a synchronous communication. If in case, the [LocalServer](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/LocalHubNode/LocalServer.py) is stopped manually, repeat from steps 3. If we stop [CentralServer](https://github.com/VenkataKanumuri/Privacy-Preserving-Localization/tree/main/FinalCode/CentralNode/CentralServer.py), then we have to stop every program and follow the instructions from initial step.


**Results:** 
When we ran the filter on 4 nodes with random movement of laser pointer below figure was the resulting plot

<img  src="IMAGES/resultPlot.jpeg" alt="result plot" width="600"/>

We covered the cameras with a transparent tape and ran the filter on a fixed path (along the tape placed on the floor) to see the effect when there is a noise (as in below images)

<img  src="IMAGES/IMG_0588.jpeg" alt="frame image" width="600"/>

<img  src="IMAGES/IMG_0587.jpeg" alt="frame image 2" width="600"/>

<img  src="IMAGES/blurplot.jpeg" alt="plot" width="600"/>


We also ran the filter without the tape (considering no noise) on the same path and plotted the below graph

<img  src="IMAGES/withfilter.jpeg" alt="plot1" width="600"/>

Finally, to recreate the results please follow the above mentioned steps (Instructions to run the setup) and can verify with plots.

***Additional:***
To ensure the proper functioning of Encrypted Multisensor Information Filter, we also simulated theoretical measurements.
You can find the code of simulation in [Simulation_code](https://code.ovgu.de/mushunur/pploc/-/tree/master/Simulation_code) folder
To run the simulation code, please follow the instructions specified in [Simulation document](https://code.ovgu.de/mushunur/pploc/-/blob/master/Code_Doc.pdf)




