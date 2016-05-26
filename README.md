# Metlink.org.nz Unofficial API Description

This is an unofficial description of the API used by https://www.metlink.org.nz/
to provide current bus, train and ferry information.

I wanted to write a little Slack bot the other day to tell me when the next train would be at the J'ville mall. I could look at the web site--it's a very nice web site--but I wanted to do this instead.


# MetLink API v1 Analysis

## Images and Links

`Icon` and `Link` references have `\` escaped /'s. I don't know why this is but I'm sure there's some reason.

These point to resources which are available under https://www.metlink.org.nz/

Example: `"Icon": "\/assets\/StopImages\/WELL.jpg"`


## `/api/v1/StopDepartures/<STOP>`

Examples:

- `https://www.metlink.org.nz/api/v1/StopDepartures/WELL`
- `https://www.metlink.org.nz/api/v1/StopDepartures/CROF`
- `https://www.metlink.org.nz/api/v1/StopDepartures/7093`

The `StopDepartures` call returns at nice summary of upcoming departures along with information about a specific stop.

The returned JSON object contains 4 top level items:

- `LastModified` - an RFC3339 formated timestamp related to this query. Ex: `2016-05-26T14:04:47+12:00`

- `Stop` - a record containing static details about this stop

```
  "Stop": {
    "Name": "Wellington Station",
    "Sms": "WELL",
    "Farezone": "1",
    "Lat": "-41.2789686",
    "Long": "174.7805617",
    "LastModified": "2015-09-03T11:14:30+12:00",
    "Icon": "\/assets\/StopImages\/WELL.jpg"
  }
```

- `Notices` (optional) - a list of records. The `LineNote` fields contain the interesting info.

```
  "Notices": [
    {
      "RecordedAtTime": "2016-05-26T14:04:39+12:00",
      "MonitoringRef": "7093",
      "LineRef": "",
      "DirectionRef": "",
      "LineNote": "Police incident resolved in the CBD. Expect some delays while services, esp in & out of Karori, get back to normal. metlink.org.nz"
    }
  ]
```

- `Services` - a list of (always 20?) upcoming trains or buses coming to this stop, soonest first


Example of a `Services` record list item:

```
    {
      "ServiceID": "JVL",
      "IsRealtime": true,
      "VehicleRef": "3718",
      "Direction": "Outbound",
      "OperatorRef": "RAIL",
      "OriginStopID": "WELL",
      "OriginStopName": "Wellington Stn",
      "DestinationStopID": "JOHN",
      "DestinationStopName": "JOHN - All stops",
      "AimedArrival": "2016-05-26T14:09:00+12:00",
      "AimedDeparture": "2016-05-26T14:09:00+12:00",
      "VehicleFeature": null,
      "DepartureStatus": "delayed",
      "ExpectedDeparture": "2016-05-26T14:11:28+12:00",
      "DisplayDeparture": "2016-05-26T14:11:28+12:00",
      "DisplayDepartureSeconds": 479,
      "Service": {
        "Code": "JVL",
        "TrimmedCode": "JVL",
        "Name": "Johnsonville Line (Johnsonville - Wellington)",
        "Mode": "Train",
        "Link": "timetables\/train\/JVL"
      }
    },
```

## `/api/v1/ServiceLocation/<SERVICE>`

Examples:

- `https://www.metlink.org.nz/api/v1/ServiceLocation/JVL` (Jville train)
- `https://www.metlink.org.nz/api/v1/ServiceLocation/14` (14 bus)

The returned JSON object contains 2 top level items:

- `LastModified` - an RFC3339 formated timestamp related to this query. Ex: `2016-05-26T14:04:47+12:00`

- `Services` - A list of major stops. For the Jville train, for example, it lists Wellington and Johnsonville only.

```
      "Services": [
        {
          "RecordedAtTime": "2016-05-26T14:35:32+12:00",
          "VehicleRef": "3729",
          "ServiceID": "JVL",
          "HasStarted": true,
          "DepartureTime": "2016-05-26T14:32:00+12:00",
          "OriginStopID": "WELL",
          "OriginStopName": "Wellington Stn",
          "DestinationStopID": "JOHN",
          "DestinationStopName": "Johnsonville Stn",
          "Direction": "Outbound",
          "Bearing": "36",
          "BehindSchedule": true,
          "VehicleFeature": null,
          "DelaySeconds": 82,
          "Lat": "-41.2633438",
          "Long": "174.7850647",
          "Service": {
            "Code": "JVL",
            "TrimmedCode": "JVL",
            "Name": "Johnsonville Line (Wellington - Johnsonville)",
            "Mode": "Train",
            "Link": "timetables\/train\/JVL"
          }
        },
        {
          "RecordedAtTime": "2016-05-26T14:35:32+12:00",
          "VehicleRef": "3736",
          "ServiceID": "JVL",
          "HasStarted": true,
          "DepartureTime": "2016-05-26T14:30:00+12:00",
          "OriginStopID": "JOHN",
          "OriginStopName": "Johnsonville Stn",
          "DestinationStopID": "WELL",
          "DestinationStopName": "Wellington Stn",
          "Direction": "Inbound",
          "Bearing": "204",
          "BehindSchedule": false,
          "VehicleFeature": null,
          "DelaySeconds": -2,
          "Lat": "-41.2420845",
          "Long": "174.7938232",
          "Service": {
            "Code": "JVL",
            "TrimmedCode": "JVL",
            "Name": "Johnsonville Line (Johnsonville - Wellington)",
            "Mode": "Train",
            "Link": "timetables\/train\/JVL"
          }
        }
      ]
```

# Working Software


## Go

If you happen to have a Go compiler installed then try this:

```
$ go build -o stopstat go/stopstat/main.go 

$ ./stopstat -stop 7093
Taiaroa Street (near 10)
Notices:
    Some cancelled services & CLOSED CBD bus stops for Massey Uni Pde, 1-1.30pm approx. Thurs 26 May. metlink.org.nz
Services:
    6:43AM  Strathmore Park - Outbound - Wgtn Station
    6:59AM  Strathmore Park - Outbound - Khandallah
    7:13AM  Strathmore Park - Outbound - Molesworth Street
    7:19AM  Strathmore Park - Outbound - Khandallah
    7:28AM  Strathmore Park - Outbound - Wgtn Station
    7:43AM  Strathmore Park - Outbound - Molesworth Street
    7:53AM  Strathmore Park - Outbound - Wgtn Station
    7:55AM  Strathmore Park - Inbound - School Bus
    7:59AM  Strathmore Park - Outbound - Khandallah
    8:08AM  Strathmore Park - Outbound - Victoria University
    8:30AM  Strathmore Park - Outbound - Victoria University
    8:59AM  Strathmore Park - Outbound - Khandallah
    9:29AM  Strathmore Park - Outbound - Khandallah
    9:59AM  Strathmore Park - Outbound - Khandallah
    10:29AM  Strathmore Park - Outbound - Khandallah
    10:59AM  Strathmore Park - Outbound - Khandallah
    11:29AM  Strathmore Park - Outbound - Khandallah
    11:59AM  Strathmore Park - Outbound - Khandallah
    12:29PM  Strathmore Park - Outbound - Khandallah
    12:59PM  Strathmore Park - Outbound - Khandallah
```

## Python

soon

## Slack Bot (in Go)

I have a working Slack bot which I'll tidy up and drop here.
