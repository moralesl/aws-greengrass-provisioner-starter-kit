# Starter kit for aws-greengrass-provisioner
Starter kit for [aws-greengrass-provisioner](https://github.com/awslabs/aws-greengrass-provisioner).

## Overview
The described steps will setup:
1. Greengrass Core software on the Edge, install its dependencies and download the Greengrass group certificates
1. Deploy the Lambda function to the Greengrass group

For more details about this setup take a look in the [Basic Tutorial](https://github.com/awslabs/aws-greengrass-provisioner/blob/master/docs/BasicTutorials.md) from the [aws-greengrass-provisioner](https://github.com/awslabs/aws-greengrass-provisioner).

For more inspiration on how to setup Greengrass functions, check out [aws-samples/aws-greengrass-lambda-functions](https://github.com/aws-samples/aws-greengrass-lambda-functions) repository.

## Requirements
To use GGP you need to have this software packages:
- An AWS account
- [the AWS Command Line Interface installed][aws1]
- [the AWS Command Line Interface configured][aws2]
- [Docker installed][docker]

[aws1]: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html
[aws2]: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html
[docker]: https://docs.docker.com/install/overview/

To deploy the starter kit to a Raspberry PI you need:
- 1 (or more) Raspberry Pi with Raspbian OS installed and pingable
- [SSH enabled][ssh] on the Raspberry Pi
- IP of Raspberry Pi, assigned to the bash variable `$RPI_IP`

[ssh]: https://www.raspberrypi.org/documentation/remote-access/ssh/

## Usage
Go to the folder where you git cloned the repository and in the console, run:

```bash
./ggp.sh -g RaspberryPiGGP -a ARM64 -d deployments/python3-hello-world.conf --script
```

With this command, you provide to the GGP:

- The new Greengrass group name you want to create: `RaspberryPiGGP`
- The platform of the Edge that you want to setup: `ARM64` (i.e. Raspberry Pi)
- The Lambda function that you want to create: `python3-hello-world`
- The flag `--script` signaling that you want a *1-click bootstrap setup* script for the Edge

Now, you are ready to *ssh copy* the setup script on one of your Raspberry Pis and execute it. Make sure that the Raspberry Pi is turned on, connected to the same Wi-Fi network, pingable, has shh enabled and then run:

```bash
cd build/
scp gg.RaspberryPiGGP.sh pi@$RPI_IP:~
ssh pi@$RPI_IP "~/gg.RaspberryPiGGP.sh"
```

The bootstrap script will ask you the following three questions:

```bash
$ Install Greengrass? #This unpacks Greengrass and puts all of the files into the /greengrass directory
y
$ Start Greengrass? #This runs ./start.sh to start Greengrass when the installation is complete
y
$ Update dependencies? #This installs all of the dependencies for Greengrass
y
```

Answer y to all three questions.

> Depending on the Raspberry Pi you are using you might get the following message _"You must reboot and re-run this installer run sudo reboot. Wait for the device to restart, ssh back in, and run the installer script again."_

You can now perform the same steps for another RaspberryPi.

## Verify successful deployment
### On the Raspberry Pi

When the script is finished and Greengrass starts it will pull down your deployment of the `python3-hello-world`. Your console will be monitoring the Greengrass logs at this point. You can CTRL-C out of it if you need to get back to the system. You can start the monitoring again by running ./monitor.sh.

After a successful deployment, the last four lines you see in the console should look like:

```bash
[2018-01-17T21:14:01.318Z][INFO]-Trying to subscribe to topic $aws/things/RaspberryPiGGP_Core-gda/shadow/update/delta
[2018-01-17T21:14:01.355Z][INFO]-Subscribed to : $aws/things/RaspberryPiGGP_Core-gda/shadow/update/delta
[2018-01-17T21:14:01.355Z][INFO]-Trying to subscribe to topic $aws/things/RaspberryPiGGP_Core-gda/shadow/get/accepted
[2018-01-17T21:14:01.422Z][INFO]-Subscribed to : $aws/things/RaspberryPiGGP_Core-gda/shadow/get/accepted
```

### On the AWS IoT Console

On the AWS IoT console on AWS, subscribe to the `+/+/hello/world` topic and you should see messages showing up every 5 seconds like this:

```json
{
  "group_id": "4602bcec-8921-4204-8ea6-d79b6588204a",
  "thing_name": "RaspberryPiGGP_Core",
  "thing_arn": "arn:aws:iot:eu-central-1:xxxxxxxxxxxx:thing/RaspberryPiGGP_Core",
  "message": "Hello world! Sent from Greengrass Core from Python 3.7.3 running on platform Linux-4.19.118-v7+-armv7l-with-debian-10.4 4602bcec-8921-4204-8ea6-d79b6588204a RaspberryPiGGP_Core arn:aws:iot:eu-central-1:xxxxxxxxxxxx:thing/RaspberryPiGGP_Core"
}
```

## Acknowledgement
This repository is a simplified version of [aws-samples/aws-greengrass-lambda-functions](https://github.com/aws-samples/aws-greengrass-lambda-functions).
