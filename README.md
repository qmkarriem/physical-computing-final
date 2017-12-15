# physical-computing-final
#### Names: Quran Karriem, Rebecca Uliasz
#### Date: December 8, 2017
## Project: SynthBall
### Conceptual Description
We’ve created two devices that each incorporate inertial sensors to transmit data about their individual movement in real time; specifically, elastic/silicon balls that can be thrown, caught, rolled and bounced. The devices possess their own type of temporality, where human gestures performed while in contact with them are automatically recorded, and used to inform and manipulate the datastream as it unfolds in real time, thus acting as a type of mnemonic device of gesture. The data transmitted will include three dimensional accelerometer data, a gyroscope to measure changes in rotational orientation, a magnetometer to measure gravitational fields (like the Earth’s). The data will be streamed to media software to control aspects of audio and visuals in real-time. 

### Form
Our goal was to remediate physical motion, performance and play from material bodies and objects to abstract digital imagery and sound. In using motion capture (or, more accurately “motion streaming”) to create non-representational transformations, we’re interested in exploring whether elements of gesture or temporal meaning can be dissociated from human form. We’ve chosen the spherical form factor because it is ancient and universally associated with physical play; we share an interest in constructing virtual play spaces that augment and inform, rather than replace, physical experiences as a challenge to the wholly digitized perceptual systems seen in virtual reality products. The objects could be further used to model and manipulate social dynamics. How might they reconfigure the ways that people with in a space act together and alone?

### Code
The function of the code running on the Photon is primarily to read the IMU and transmit the data over UDP. The printGyro(), printAccel(), printMag() functions are largely identical, but handle the data from the different sensors in the LSM9DS1 and run on each iteration of the loop() and are the means of data transmission for each of the sets of sensor readings:

```c++ 
void printGyro()
{
  imu.readGyro();
  int ret = snprintf(buffer, bufferSize, "G: %f %f %f", imu.calcGyro(imu.gx), imu.calcGyro(imu.gy), imu.calcGyro(imu.gz));

  if (udp.sendPacket(buffer, bufferSize, remoteIP, remotePort) >= 0) {
    // Success
    #ifdef SERIAL_DEBUG
      Serial.printlnf("%d", buffer);
    #endif
  }
  else {
    #ifdef SERIAL_DEBUG
      Serial.printlnf("send failed");
    #endif
            // On error, wait a moment, then reinitialize UDP and try again.
    delay(1000);
    udp.begin(0);
  }
  Serial.print("G: ");
#ifdef PRINT_CALCULATED
  Serial.print(imu.calcGyro(imu.gx), 2);
  Serial.print(" ");
  Serial.print(imu.calcGyro(imu.gy), 2);
  Serial.print(" ");
  Serial.println(imu.calcGyro(imu.gz), 2);
  //Serial.println(" deg/s");
#elif defined PRINT_RAW
  Serial.print(imu.gx);
  Serial.print(", ");
  Serial.print(imu.gy);
  Serial.print(", ");
  Serial.println(imu.gz);
#endif
}
```

We use Particle.subscribe() to update the IP address and port the Photon transmits data to without needing to flash new code to it. We publish events manually from the Particle cloud console to update IP/port, with a unique topic for each Photon:

```c++
// receive a new port from Particle cloud Terminal
void updateRemotePort(const char *event, const char *data) {
  remotePort = atoi(data);
  Particle.publish("remotePortCallback", remotePort);
}

//receive a new IP address from Particle cloud Terminal
void updateRemoteIP(const char *event, const char *data) {
  unsigned char IPHandler[4] = {0}; //need to parse into . separated values
  size_t index = 0;
  while (*data){
    if (isdigit((unsigned char)*data)){
      IPHandler[index] *= 10;
      IPHandler[index] += *data - '0';
    } else {
      index++;
    }
    data++;
  }
  sprintf(IPString, "%i, %i, %i, %i", IPHandler[0], IPHandler[1], IPHandler[2], IPHandler[3]);
  remoteIP = IPHandler;
  Particle.publish("remoteIPCallback", IPString);
  Particle.publish("remoteIPCallback", String(remoteIP));
}
```
We briefly also used the SparkFunMAX17043 library to report battery life along with IMU data. It worked well when all components were functioning, but caused tremendous slowdown in data transmission speed when we experienced issues with the fabrication process that may have shorted out the reporting, but not charging functions of the battery shield.
 


**Finished Enclosure**
The finished enclosure was created through a silicone moulding process. We first designed and printed a 3D mold. We used a household silicone that we treated with glycerin to expedite the drying process. The components were placed in plastic for security and embedded inside the mold while it was still plyable. 

The moulding process:
![3d mold](https://github.com/qmkarriem/physical-computing-final/blob/master/images/IMG_5508.JPG)
![dish soap](https://github.com/qmkarriem/physical-computing-final/blob/master/images/IMG_5510.JPG)
![silicone](https://github.com/qmkarriem/physical-computing-final/blob/master/images/IMG_5511.JPG)
![silicone](https://github.com/qmkarriem/physical-computing-final/blob/master/images/IMG_3799.JPG) 
![silicone2](https://github.com/qmkarriem/physical-computing-final/blob/master/images/IMG_3801.JPG)

Becca with the end product:
![Becca](https://github.com/qmkarriem/physical-computing-final/blob/master/images/IMG_3819.JPG)

**Electronics Exposed**
![photon and bag](https://github.com/qmkarriem/physical-computing-final/blob/master/images/IMG_5504.JPG)
![bad shield](https://github.com/qmkarriem/physical-computing-final/blob/master/images/IMG_5505.JPG)
![photon](https://github.com/qmkarriem/physical-computing-final/blob/master/images/IMG_5506.JPG)
![IMU shield](https://github.com/qmkarriem/physical-computing-final/blob/master/images/IMG_5507.JPG)

### Technical Details
Hardware Used:
* Particle Photon 
* IMU Sensors and shield 
* Sparkfun battery shield
* Silicone and 3d printed mold

**Other Materials**
* Rubberbands
* Ziplock Bags
* Dishwashing Liquid (Blue)
* Water

### End Goal:
We anticipate uses of this project in live performances, installations and participatory game-based encounters with observers. The objects themselves are being created as part of our coursework in this semester’s Physical Computing course, and similar devices (wearable versions rather than the ball form here) and software will be used in a new dance work that will debut at MoogFest in Spring 2018. 

Our reach goal for this semester was to have 4 device prototypes fabricated for testing, with custom designed PCBs and range finding capabilities. The "feasible goal" we set was to have at least 2 fabricated and functioning. We met the latter goal, but were not able to fabricate custom PCBs, implement rangefinding, or create a set of 4 devices. We've been encouraged by faculty to submit the devices to Duke I&E for patenting and potential funding for future development.
