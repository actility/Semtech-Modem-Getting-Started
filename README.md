
# Evaluating Semtech modem using ThingPark Wireless

RaphaelApfeldorfer-Actility edited this page on Oct 21th, 2019

## Introduction

This guide will direct you through the process of getting started with developing a secure LoRa end device product using Semtech modem along with ThingPark Wireless Network Server DEV1 for partners

This guide describes the following:

  1.  [Get a ThingPark Partner account](#get-a-partner-account-for-thingPark-wireless)
  1.  [Collect the Device identifiers](#collect-the-device-identifiers)
  1.  [Claim device in Semtech JS](#claim-device-in-semtech-join-server)
  1.  [Provision/Activate device in ThingPark Wireless](#provisionactivate-device-in-thingpark-wireless)


## Get a partner account for ThingPark Wireless
To request an account on ThingPark Wireless that can operate with Semtech modem, post the following email
```
To: partner-activation@actility.com
Subject: ThingPark Activation evaluation
Mail: Please provide me a free test account for ThingPark Activation.
Account Name: Paul Smith
Company:      Acme
Login email:  paul.smith@acme.com
Project:      Gas meter in Europe
Timeline:     Mass production end of 2020
URL:          https://www.acme.com/products/gasmeter.html
```
Please anticipate your request as it may requires a few days to be processed.

## Collect the Device identifiers
In order to commission a Semtech modem in ThingPark Wireless, the following idenfiers are required:
* ***DevEUI***: LoRaWAN device 64-bit unique identifier assigned by the Device manufacturer (or using Secure Element default value)
* ***JoinEUI***: LoRaWAN JS 64-bit unique identifier of the Join Server on which AppKey of the device is stored
* ***PIN code***: Device secret code allowing to claim device in Semtech JS

Connect to the Semtech modem evaluation kit with serial port at ***baud rate***=`115200 bps`, and trigger the Evaluation mode using the following sequence:
1. Press and hold down the User button (blue) on the Nucleo board.
1. While holding down the User button, press and release the Reset button (black)
1. Continue to hold down User button for at least 5 seconds and release the User button

Then input the following commands
```
>------- Evaluation Mode ---------<

 Type 'h' to view available commands

> GetDevEui
> GetJoinEui
> GetPin
```
Please refer to Semtech Modem User Guide for more details.
 
## Provision device in Semtech Join Server

### Claim device in Semtech Join Server
Go on your [Semtech Developer Portal JS account](https://www.loracloud.com/portal/join_service/)

Click on _Network Servers_

Select ***Network Server***=`Actility ThingPark Wireless DEV1` and press Add, and _Save networks Servers_

<img src=resources/AddNS.gif alt="Add Actility NS" width="600"/>

Click on _Devices_

<img src=resources/SemtechJS.gif alt="Semtech device" width="600"/>

Select Claim Individual device and input DevEUI and PIN code previously retrieved

<img src=resources/ClaimDevice.gif alt="Claim device" width="600"/>

### Set Wrapping keys for AppSKey

***AppSKey*** might be delivered to Network Server (and then Application Server) in clear text or encrypted with a Key Encryption Key.
In order to setup how AppSKey is delivered, go to _Keys and Credentials_

<img src=resources/SetupKEK.gif alt="Setup KEK" width="600"/>

By default, AppSKey is delivered in clear text (no wrapping key). You can use this default but ThingPark Wireless supports any type.

## Provision/Activate device in ThingPark Wireless

### Create Application Server and routing profile
 
To create a new Application Server routing profile, first create a local Application Server. 

Click on _Application Servers_, _Add Application servers_ -> Create and select ***Type***=`HTTP Application Server (LoRaWAN)`

<img src=resources/CreateAS.gif alt="Create AS" width="600"/>

Select ***Content Type*** according to your Application Server requirement, but note that the Application Server **must** support end-to-end encryption (see [Activate device](#activate-device)).
Then add the destination route by clicking Route -> Add.

<img src=resources/CreateASroute.gif alt="Create AS route" width="600"/>

Once the Application Server is created, create an Application Server routing profile. Click on _AS routing profiles_ -> Create and select ***Type***=`Local application server` and your ***Destination***.

<img src=resources/CreateASRP.gif alt="Create AS routing profile" width="600"/>

### Provision device in Network Server
Login your ThingPark DEV1 partner account and open [Device Manager](https://dev1.thingpark.com/deviceManager)
The device is provisioned as usual, except no AppKey needs to be provided to the Network Server

Click on Add Device -> Create

<img src=resources/DeviceDM.gif alt="Create Device" width="600"/>

Select ***Manufacturer*** = `Generic` and your ***LoRaWAN device profile***, ***activation type*** and fill in the ***DevEUI***/***AppEUI***retrieved previously (note that AppEUI is the JoinEUI previously retrieved, which was the initial name in specifications earlier than LoRaWAN1.0.3).

It is mandatory to select a ***Connectivity Plan*** and an ***Application Server routing Profile*** for your device to be fully provisioned and ready to be activated.

<img src=resources/CreateDeviceDM.gif alt="Device creation fields" width="600"/>

### Activate device 
The device is now ready to be activated
LoRaWAN data can be monitored using [Wireless Logger](https://dev1.thingpark.com/wLogger)

Note that data is shown encrypted in Wireless Logger and the payload is delivered to the Application Server encrypted with the AppSKey.

If your Application Server does not support end-to-end security, you can decode payload manually. 

First, send data to an HTTP capture service such as [hookbin](https://hookbin.com/). Copy the endpoint address in destination of the Application Server created in [Create Application Server](#create-application-server-and-routing-profile), trigger an uplink on the board and refresh the Hookbin page:

<img src=resources/hookbin.gif alt="Retrieve link from Hookbin" width="600"/>

In <DevEUI_uplink> document, the applicative payload is present in ***<payload_hex>*** and the AppSKey in <***AppSKey***>. AppSKey is either in clear or encrypted depending on the setting selected in Semtech JS in [Set Wrapping keys for AppSKey](#set-wrapping-keys-for-appskey).

Using pyThingPark package (available on Python 3 only, install with `pip3 install pyThingPark`), you can easily decode the "DevEUI_uplink" frame using the following code:

        >>> hookbinUL = '{"DevEUI_uplink": {"Time": "2019-10-22T17:55:21.281+02:00","DevEUI": "0016C001FF0000A7","FPort": 102,"FCntUp": 3,"ADRbit": 1,"MType": 2,"FCntDn": 0,"payload_hex": "6b349c7db3","mic_hex": "88319eb6","Lrcid": "00000127","LrrRSSI": -59.000000,"LrrSNR": 9.500000,"SpFact": 7,"SubBand": "G0","Channel": "LC3","DevLrrCnt": 1,"Lrrid": "C0001565","Late": 0,"LrrLAT": 0.000000,"LrrLON": 0.000000,"Lrrs": {"Lrr": [{"Lrrid": "C0001565","Chain": 0,"LrrRSSI": -59.000000,"LrrSNR": 9.500000,"LrrESP": -59.461838}]},"CustomerID": "100118249","CustomerData": {"alr":{"pro":"LORA/Generic","ver":"1"}},"ModelCfg": "0","AppSKey": "6196ceaf3eb5f44921e47717f68f8d66","InstantPER": 0.000000,"MeanPER": 0.000000,"DevAddr": "042910AB","AckRequested": 0,"rawMacCommands": "","TxPower": 16.000000,"NbTrans": 1}}'
        >>> from pyThingPark import uplinkTunnel 
        >>> uplink = uplinkTunnel.DevEUI_uplink(hookbinUL)
        >>> clearPayload = uplink.decryptPayload(ASTK=None)
        >>> bytearray.fromhex(clearPayload).decode()
        'HELLO'

## Support
Should you need support or if you have any feedback, please contact Actility support by entering a support case request at partner@actility.com.