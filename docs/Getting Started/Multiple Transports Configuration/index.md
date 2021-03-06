
# Setting Up Multiple Transports

The Multiple Transports feature allows apps connected to SDL Core to start another connection over a different transport for certain services. For example, an app connected over Bluetooth can use WiFi as a Secondary Transport for video streaming. This guide will walk you through how to configure the Multiple Transports feature using the `smartDeviceLink.ini` file.

## Initial Setup

Modify the following lines in `smartDeviceLink.ini`.

- To enable Multiple Transports in Core:

```
[MultipleTransports]
...
MultipleTransportsEnabled = true
```

- To set the available Secondary Transport types for a given Primary Transport:

```
[MultipleTransports]
...
SecondaryTransportForBluetooth = WiFi
;SecondaryTransportForUSB =
;SecondaryTransportForWiFi =
```

!!! NOTE   
The values which can be used in the `SecondaryTransportFor` configuration are `WiFi`, `Bluetooth` and `USB`  
!!!


## Audio and Video Streaming

Modify the services map in `smartdeviceLink.ini` to restrict video and audio streaming services to specific transport types.

```
[ServicesMap]
...
AudioServiceTransports = TCP_WIFI
VideoServiceTransports = TCP_WIFI, AOA_USB
```
- Transports are listed in preferred order
- If a transport is not listed, then the service is not allowed to run on that transport
- If the `AudioServiceTransports`/`VideoServiceTransports` line is omitted, the corresponding service will be allowed to run on the Primary Transport

### Secondary Transport Types

|String|Type|Description|
|------|----|-----------|
|IAP_BLUETOOTH|Bluetooth|iAP over Bluetooth|
|IAP_USB_HOST_MODE|USB|iAP over USB, and the phone is running as host|
|IAP_USB_DEVICE_MODE|USB|iAP over USB, and the phone is running as device|
|IAP_USB|USB|iAP over USB, and Core cannot distinguish between Host Mode and Device Mode|
|IAP_CARPLAY|WiFi|iAP over Carplay wireless|
|SPP_BLUETOOTH|Bluetooth|Bluetooth SPP, either legacy SPP or SPP multiplexing|
|AOA_USB|USB|Android Open Accessory|
|TCP_WIFI|WiFi|TCP connection over Wi-Fi|

## Resources

For more information on how the Multiple Transports feature works, see the [Feature Documentation](../../feature-documentation/multiple-transports).
