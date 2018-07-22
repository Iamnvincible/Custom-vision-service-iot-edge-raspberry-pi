---
services: iot-edge, custom-vision-service, iot-hub
platforms: python
author: ebertrams
---

# Custom Vision + Azure IoT Edge on a Raspberry Pi 3

## Get started

0. Clone this repo `git clone https://github.com/Iamnvincible/Custom-vision-service-iot-edge-raspberry-pi.git`
1. Create an Azure Iot Hub
2. Create an IoT Edge device in your Iot Hub, and note your device's **Connection String** for further use.
3. Install Azure IoT Edge on RaspberryPi(ARM32v7/armhf), you can visit [here](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge-linux-arm) for more infomation. Simply, you can just follow my steps.

    Install Docker,run  `wget -qO- https://get.docker.com/ | sh`.
    Install&Configure IoT Edge Security Daemon, please follow the steps at [here](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge-linux-arm#install-the-iot-edge-security-daemon).
4. Navigate to your device detail page on Azure Portal, you may see two modules, named $edgeAgent and $edgeHub.
5. Build Container

Go to Custom-vision-service-iot-edge-raspberry-pi/modules/CameraCapture/app, open `main.py` file and add your device's connection string at line 129. And replace `raspberrypi:80/image` in filed `IMAGE_PROCESSING_ENDPOINT` with your device's real ip, such as `http://192.168.3.4`.Don't forget to save it. Then return to `CameraCapture` folder. Run this command.
```
sudo docker build -t camera -f arm32v7.Dockerfile .
```
 That will take a few minutes.

Go to ImageClassifierService folder, run a similar command.

```
sudo docker build -t classifier -f arm32v7.Dockerfile .
```

6. Test
Please open two bash windows, in one window run this command.
```
sudo docker run -p 80:80 classifier
```
In another one run this command.
```
sudo docker run camera
```
You can see log printed in both windwos.

7. Kill containers
```
sudo docker ps
```
Note the container id listed.
The use this command to kill container.
```
sudo docker kill <containid>
```

8. Create ACR in your Azure portal and turn admin mode, note your username and password.

9. Push images to ACR
Login.
```
sudo docker login <your_registry>.azurecr.io -u username -p password
```
Tag image.
```
sudo docker tag camera <your_registry>.azurecr.io/camera
```
Push image.
```
sudo docker push <your_registry>.azurecr.io/camera
```
Similarly, push another one.
```
sudo docker tag classifier <your_registry>.azurecr.io/classifier

sudo docker push <your_registry>.azurecr.io/classifier
```
10. Set modules

Go to your IoT device's page, click `set modules` button, set your ACR info first, and add an IoT Edge Module, fill the image url with `<your_registry>.azurecr.io/classifier` and in the `Container Create Options
` cell fill it with 
```
{"HostConfig":{"PortBindings":{"80/tcp":[{"HostPort": "80"}]}}}
```
click next and then submit.

And now add the camera module, similarly, fill the image url with `<your_registry>.azurecr.io/camera` and submit it.

A few minutes later, you can see your modules in device page with status `running`.

Also you can use command `sudo iotedge list` on your device to see modules' status.

11. Monitor D2C message

If you installed 'Azure IoT Hub` extension in you VS Code, you can monitor Device to Cloud message easily.

### Original README
> [!NOTE]
> This sample still uses the IoT Edge Preview bits. To use it with the new IoT Edge GA bits, you will need to update the **Camera capture** and **SenseHat display** modules manually to use the latest IoT SDK versions. The **Custom vision** should remain the same. 

This is a sample showing how to deploy a Custom

 Vision model to a Raspberry Pi 3 device running Azure IoT Edge. This solution is made of 3 modules:

- **Camera capture** - this module captures the video stream from a USB camera, sends the frames for analysis to the custom vision module and shares the output of this analysis to the edgeHub. This module is written in python and uses [OpenCV](https://opencv.org/) to read the video feed.
- **Custom vision** - it is a web service over HTTP running locally that takes in images and classifies them based on a custom model built via the [Custom Vision website](https://azure.microsoft.com/en-us/services/cognitive-services/custom-vision-service/). This module has been exported from the Custom Vision website and slightly modified to run on a ARM architecture. You can modify it by updating the model.pb and label.txt files to update the model.
- **SenseHat display** - this module gets messages from the edgeHub and blinks the raspberry Pi's senseHat according to the tags specified in the inputs messages. This module is written in python and requires a [SenseHat](https://www.raspberrypi.org/products/sense-hat/) to work.

## Get started
1- Update the module.json files of the 3 modules above to point to your own Azure Container Registry

2- Build the full solution by running the `Build IoT Edge Solution` command from the [Azure IoT Edge extension in VS Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge).

## Prerequisites

You can run this solution on either of the following hardware:

- **Raspberry Pi 3**: Set up Azure IoT Edge on a Raspberry Pi 3 ([instructions](https://blog.jongallant.com/2017/11/azure-iot-edge-raspberrypi/)) with a [SenseHat](https://www.raspberrypi.org/products/sense-hat/) and use the arm32v7 module tags.

- **Simulated Azure IoT Edge device** (such as a PC): Set up Azure IoT Edge ([instructions on Windows](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-simulate-device-windows), [instructions on Linux](https://docs.microsoft.com/en-us/azure/iot-edge/tutorial-simulate-device-linux)) and use the amd64 module tags. A test x64 deployment manifest is already available. To use it, rename the 'deployment.template.test-amd-64' to 'deployment.template.json', then build the IoT Edge solution from this manifest and deploy it to an x64 device.
