---
title: Sensor partner integration
description: This article describes sensor partner integration.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
---

# Sensor partner integration

This article provides information about the Azure FarmBeats Translator component, which enables sensor partner integration.

Using this component, partners can develop sensors that integrate with FarmBeats to use the API and send customer device data and telemetry to FarmBeats Datahub. Data is visualized by using the FarmBeats Accelerator. Data can be used for data fusion and for building machine learning/artificial intelligence models.

## Link a FarmBeats account

After you've purchased and deployed devices or sensors, you can access the device data and telemetry on your device partners' software as a service (SaaS) portal. Device partners enable you to link your account to your FarmBeats instance on Azure. The following credentials must be filled in by you or your system integrator:

   - Display name (an optional field for users to define a name for this integration)
   - API endpoint
   - Tenant ID
   - Client ID
   - Client secret
   - EventHub connection string
   - Start date

   > [!NOTE]
   > The start date enables the historical data feed, that is, the data from the date specified by the user.

## Unlink FarmBeats

You have the ability to unlink an existing FarmBeats integration. Unlinking FarmBeats shouldn't delete any device or sensor metadata that was created in your data hub. Unlinking does the following:

   - Stops telemetry flow.
   - Deletes and erases the integration credentials on the device partner.

## Edit FarmBeats integration

You can edit the FarmBeats integration settings if the client secret or connection string changes. In this case, you can edit only the following fields:

   - Display name (if applicable)
   - Client secret (should be displayed in "2x8***********" format or the Show/Hide feature rather than clear text)
   - Connection string (should be displayed in "2x8***********" format or Show/Hide feature rather than clear text)

## View the last telemetry sent

You can view the timestamp of the last telemetry that was sent, which is found under **Telemetry Sent**. This is the time at which the latest telemetry was successfully sent to FarmBeats.

## Translator development

**REST API-based integration**

Sensor data integration capabilities of FarmBeats are exposed via the REST API. Capabilities include metadata definition, device and sensor provisioning, and device and sensor management.

**Telemetry ingestion**

The telemetry data is mapped to a canonical message that's published on Azure Event Hubs for processing. Azure Event Hubs is a service that enables real-time data (telemetry) ingestion from connected devices and applications.

**API development**

The APIs contain Swagger technical documentation. For more information on the APIs and their corresponding requests or responses, see [Swagger](https://aka.ms/FarmBeatsDatahubSwagger).

**Authentication**

FarmBeats uses Microsoft Azure Active Directory authentication. Azure App Service provides built-in authentication and authorization support.

For more information, see [Azure Active Directory](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization).

FarmBeats Datahub uses bearer authentication, which needs the following credentials:
   - Client ID
   - Client secret
   - Tenant ID

Using these credentials, the caller can request an access token. The token needs to be sent in the subsequent API requests, in the header section, as follows:

```
headers = {"Authorization": "Bearer " + access_token, …} 
```

The following sample Python code gives the access token, which can be used for subsequent API calls to FarmBeats.

```python
import azure 

from azure.common.credentials import ServicePrincipalCredentials 
import adal 
#FarmBeats API Endpoint 
ENDPOINT = "https://<yourdatahub>.azurewebsites.net" [Azure website](https://<yourdatahub>.azurewebsites.net)
CLIENT_ID = "<Your Client ID>"   
CLIENT_SECRET = "<Your Client Secret>"   
TENANT_ID = "<Your Tenant ID>" 
AUTHORITY_HOST = 'https://login.microsoftonline.com' 
AUTHORITY = AUTHORITY_HOST + '/' + TENANT_ID 
#Authenticating with the credentials 
context = adal.AuthenticationContext(AUTHORITY) 
token_response = context.acquire_token_with_client_credentials(ENDPOINT, CLIENT_ID, CLIENT_SECRET) 
#Should get an access token here 
access_token = token_response.get('accessToken') 
```


**HTTP request headers**

Here are the most common request headers that need to be specified when you make an API call to FarmBeats Datahub.


**Header** | **Description and example**
--- | ---
Content-Type | The request format (Content-Type: application/<format>). For FarmBeats Datahub APIs, the format is JSON. Content-Type: application/json
Authorization | Specifies the access token required to make an API call. Authorization: Bearer <Access-Token>
Accept | The response format. For FarmBeats Datahub APIs, the format is JSON. Accept: application/json

**API requests**

To make a REST API request, you combine the HTTP (GET, POST, or PUT) method, the URL to the API service, the  Uniform Resource Identifier (URI) to a resource to query, submit data to, update, or delete, and one or more HTTP request headers. The URL to the API service is the API endpoint you provide. Here's a sample: https://\<yourdatahub-website-name>.azurewebsites.net

Optionally, you can include query parameters on GET calls to filter, limit the size of, and sort the data in the responses.

The following sample request is to get the list of devices.

```bash
curl -X GET "https://microsoft-farmbeats.azurewebsites.net/Device" -H "Content-Type: application/json" -H "Authorization: Bearer <Access-Token>"
```
Most GET, POST, and PUT calls require a JSON request body.

The following sample request is to create a device. (This sample has an input JSON with the request body.)

```bash
curl -X POST "https://microsoft-farmbeats.azurewebsites.net/Device" -H  "accept: application/json" -H  "Content-Type: application/json" -H "Authorization: Bearer <Access-Token>" -d "{  \"deviceModelId\": \"ID123\",  \"hardwareId\": \"MHDN123\",  \"reportingInterval\": 900,  \"name\": \"Device123\",  \"description\": \"Test Device 123\",}"
```

## Data format

JSON is a common language-independent data format that provides a simple text representation of arbitrary data structures. For more information, see [json.org](http://json.org).

## Metadata specifications

FarmBeats Datahub has the following APIs that enable device partners to create and manage device or sensor metadata. 

- /**DeviceModel**: DeviceModel corresponds to the metadata of the device, such as the manufacturer and the type of device, which is either gateway or node. 
- /**Device**: Device corresponds to a physical device present on the farm.
- /**SensorModel**: SensorModel corresponds to the metadata of the sensor, such as the manufacturer, the type of sensor, which is either analog or digital, and the sensor measure, such as ambient temperature and pressure.
- /**Sensor**: Sensor corresponds to a physical sensor that records values. A sensor is typically connected to a device with a device ID.
  
  **DeviceModel** |  |
  --- | ---
  Type (node, gateway)  | 1 Star |
  Manufacturer  | 2 Star |
  ProductCode  | Device product code or model name or number. For example, EnviroMonitor#6800. |
  Ports  | Port name and type, which is digital or analog.  |
  Name  | Name to identify resource. For example, model name or product name. |
  Description  | Provide a meaningful description of the model. |
  Properties  | Additional properties from the manufacturer. |
  **Device** |  |
  DeviceModelId  |ID of the associated device model. |
  HardwareId   |Unique ID for the device, such as a MAC address.  |
  ReportingInterval |Reporting interval in seconds. |
  Location    |Device latitude (-90 to +90), longitude (-180 to 180), and elevation (in meters). |
  ParentDeviceId | ID of the parent device to which this device is connected. For example, if a node is connected to a gateway, the node has parentDeviceID as the gateway. |
  Name  | Name to identify the resource. Device partners need to send a name that's consistent with the device name on the device partner side. If the device name is user defined on the device partner side, the same user-defined name should be propagated to FarmBeats.  |
  Description  | Provide a meaningful description.  |
  Properties  |Additional properties from the manufacturer.  |
  **SensorModel** |  |
  Type (analog, digital)  |Mention analog or digital sensor.|
  Manufacturer  | Name of manufacturer. |
  ProductCode  | Product code or model name or number. For example, RS-CO2-N01.  |
  SensorMeasures > Name  | Name of the sensor measure. Only lowercase is supported. For measurements from different depths, specify the depth. For example, soil_moisture_15cm. This name has to be consistent with the telemetry data. |
  SensorMeasures > DataType  | Telemetry data type. Currently, double is supported. |
  SensorMeasures > Type  | Measurement type of the sensor telemetry data. Following are the system-defined types: AmbientTemperature, CO2, Depth, ElectricalConductivity, LeafWetness, Length, LiquidLevel, Nitrate, O2, PH, Phosphate, PointInTime, Potassium, Pressure, RainGauge, RelativeHumidity, Salinity, SoilMoisture, SoilTemperature, SolarRadiation, State, TimeDuration, UVRadiation, UVIndex, Volume, WindDirection, WindRun, WindSpeed, Evapotranspiration, PAR. To add more, refer to the /ExtendedType API.
  SensorMeasures > Unit | Unit of sensor telemetry data. Following are the system-defined units: NoUnit, Celsius, Fahrenheit, Kelvin, Rankine, Pascal, Mercury, PSI, MilliMeter, CentiMeter, Meter, Inch, Feet, Mile, KiloMeter, MilesPerHour, MilesPerSecond, KMPerHour, KMPerSecond, MetersPerHour, MetersPerSecond, Degree, WattsPerSquareMeter, KiloWattsPerSquareMeter, MilliWattsPerSquareCentiMeter, MilliJoulesPerSquareCentiMeter, VolumetricWaterContent, Percentage, PartsPerMillion, MicroMol, MicroMolesPerLiter, SiemensPerSquareMeterPerMole, MilliSiemensPerCentiMeter, Centibar, DeciSiemensPerMeter, KiloPascal, VolumetricIonContent, Liter, MilliLiter, Seconds, UnixTimestamp, MicroMolPerMeterSquaredPerSecond, and InchesPerHour. To add more, refer to the /ExtendedType API.
  SensorMeasures > AggregationType  | Either none, average, maximum, minimum, or StandardDeviation.
  SensorMeasures > Depth  | The depth of the sensor in centimeters. For example, the measurement of moisture 10 cm under the ground.
  SensorMeasures > Description  | Provide a meaningful description of the measurement.
  Name  | Name to identify resource. For example, the model name or product name.
  Description  | Provide a meaningful description of the model.
  Properties  | Additional properties from the manufacturer.
  **Sensor**  |  |
  HardwareId  | Unique ID for the sensor set by the manufacturer.
  SensorModelId  | ID of the associated sensor model.
  Location  | Sensor latitude (-90 to +90), longitude (-180 to 180), and elevation (in meters).
  Port > Name  |Name and type of the port that the sensor is connected to on the device. This must be the same name as defined in the device model.
  DeviceId  | ID of the device that the sensor is connected to.
  Name  | Name to identify the resource. For example, the sensor name or product name and model number or product code.
  Description  | Provide a meaningful description.
  Properties  | Additional properties from the manufacturer.

 For information on each of the objects and their properties, see [Swagger](https://aka.ms/FarmBeatsDatahubSwagger).

 > [!NOTE]
 > The APIs return unique IDs for each instance created. This ID needs to be retained by the Translator for device management and metadata sync.


**Metadata sync**

The Translator should send updates on the metadata. For example, update scenarios are change of device or sensor name and change of device or sensor location.

The Translator should have the ability to add new devices or sensors that were installed by the user post linking of FarmBeats. Similarly, if a device or sensor was updated by the user, the same should be updated in FarmBeats for the corresponding device or sensor. Typical scenarios that require updating a device or sensor are a change in a device location or the addition of sensors in a node.


> [!NOTE]
> Delete isn't supported for device or sensor metadata.
>
> To update metadata, it's mandatory to call /Get/{id} on the device or sensor, update the changed properties, and then do a /Put/{id} so that any properties set by the user aren't lost.

### Add new types and units

FarmBeats supports adding new sensor measure types and units. For more information about the /ExtendedType API, see [Swagger](https://aka.ms/FarmBeatsDatahubSwagger).

## Telemetry specifications

The telemetry data is mapped to a canonical message that's published on Azure Event Hubs for processing. Azure Event Hubs is a service that enables real-time data (telemetry) ingestion from connected devices and applications.

## Send telemetry data to FarmBeats

To send telemetry data to FarmBeats, create a client that sends messages to an event hub in FarmBeats. For more information about telemetry data, see [Sending telemetry to an event hub](https://docs.microsoft.com/azure/event-hubs/event-hubs-dotnet-standard-getstarted-send).

Here's a sample Python code that sends telemetry as a client to a specified event hub.

```python
import azure
from azure.eventhub import EventHubClient, Sender, EventData, Receiver, Offset
EVENTHUBCONNECTIONSTRING = "<EventHub Connection String provided by customer>"
EVENTHUBNAME = "<EventHub Name provided by customer>"

write_client = EventHubClient.from_connection_string(EVENTHUBCONNECTIONSTRING, eventhub=EVENTHUBNAME, debug=False)
sender = write_client.add_sender(partition="0")
write_client.run()
for i in range(5):
    telemetry = "<Canonical Telemetry message>"
    print("Sending telemetry: " + telemetry)
    sender.send(EventData(telemetry))
write_client.stop()

```

The canonical message format is as follows:

```json
{
"deviceid": "<id of the Device created>",
"timestamp": "<timestamp in ISO 8601 format>",
"version" : "1",
"sensors": [
    {
      "id": "<id of the sensor created>",
      "sensordata": [
        {
          "timestamp": "< timestamp in ISO 8601 format >",
          "<sensor measure name (as defined in the Sensor Model)>": "<value>"
        },
        {
          "timestamp": "<timestamp in ISO 8601 format>",
          "<sensor measure name (as defined in the Sensor Model)>": "<value>"
        }
      ]
    }
 ]
}
```
All key names in the telemetry JSON should be lowercase. Examples are deviceid and sensordata.

For example, here's a telemetry message:


```json
{
  "deviceid": "7f9b4b92-ba45-4a1d-a6ae-c6eda3a5bd12",
  "timestamp": "2019-06-22T06:55:02.7279559Z",
  "version" : "1",
  "sensors": [
    {
      "id": "d8e7beb4-72de-4eed-9e00-45e09043a0b3",
      "sensordata": [
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_max": 15
        },
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_min": 42
        }
      ]
    },
    {
      "id": "d8e7beb4-72de-4eed-9e00-45e09043a0b3",
      "sensordata": [
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_max": 20
        },
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_min": 89
        }
      ]
    }
  ]
}

```

## Troubleshooting and error management

**Troubleshoot option or support**

If you're unable to receive device data or telemetry in the FarmBeats instance specified, the device partner should provide support and a mechanism for troubleshooting.

**Telemetry data retention**

The telemetry data should also be retained for a predefined time period so that it can be useful in debugging or resending the telemetry if an error or data loss occurs.

**Error management or error notification**

If an error affects the device or sensor metadata or the data integration or telemetry data flow in the device partner system, you should receive a notification. A mechanism to resolve any errors should also be designed and implemented.

**Connection checklist**

Device manufacturers or partners can use the following checklist to ensure that the credentials provided by the customer are accurate:

   - Check to see whether an access token is received with the credentials that were provided.
   - Check to see whether an API call succeeds with the access token that was received.
   - Check to see whether the EventHub client connection is established.

## Next steps

For more information about the REST API, see [REST API](references-for-farmbeats.md#rest-api).
