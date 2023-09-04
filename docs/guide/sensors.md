
Sensors

In addition to manually-entered records, farmOS also provides a framework for receiving data from automated environmental sensors. The Farm Sensor module adds a Sensor asset type, which can be tracked like any other asset.

Sub-modules (like Farm Sensor: Listener) extend the Sensor asset type by providing integration with external devices. Additional modules can be built to connect to specific types of sensors, if necessary.

It is possible to assemble your own sensors with inexpensive components and send their data to farmOS without any soldering or programming.
Farm Sensor: Listener

The Farm Sensor: Listener module is a general-purpose sensor sub-module that provides a simple "Listener" sensor type. Each sensor asset that is denoted as a listener receives a unique URL with a public and private key that data can be pushed to using standard HTTP requests with JSON-encoded data. Data is stored in the database and is displayed in graphs and in table format in the sensor asset within farmOS. Data can be exported to CSV, or queried via the API (see below).

The listener module is useful for simple data streams. For more complex data, a more customized sub-module may be necessary.
Posting data to a listener

Data can be posted to the listener using a standard HTTP request.

Each sensor will have a unique URL endpoint, which contains both the public key (as part of the address), and the private key (as a URL query paramter). This can be found in the configuration settings for the sensor asset within farmOS.

It is recommended that you serve farmOS over an HTTPS connection, so that the keys are encrypted in transit.

URL example: https://myfarm.farmos.net/farm/sensor/listener/xxxxxx?private_key=yyyyyy

The endpoint expects a JSON object with name/value pairs, and an optional timestamp.

JSON example: { "timestamp": 1541519720, "value": 76.5 }

If the timestamp is omitted, farmOS will assign the data a timestamp based on the time that the request is received.

JSON without timestamp: { "value": 76.5 }

Multiple sensor values can be included in each request (if a device measures more than one metric, for example). The name given to each value can be any string of numbers and letters other than "timestamp", which is reserved. Each name/value pair will be stored in a separate row in the database.

JSON with multiple values: { "timestamp": 1541519720, "temperature": 76.5, "humidity": 60 }

The following curl command demonstrates how to post simple data to a sensor from the command-line.

curl -H "Content-Type: application/json" -X POST \
-d '{ "timestamp": 1541519720, "value": 76.5 }' \
https://myfarm.farmos.net/farm/sensor/listener/xxxxxx?private_key=yyyyyy

Pulling data from a listener

Data can also be retrived from the sensor via the same API endpoint, using a GET request instead of a POST request. The URL is the same as the URL for posting data.

URL example: https://myfarm.farmos.net/farm/sensor/listener/xxxxxx?private_key=yyyyyy

The private key must be included, unless public API read access is allowed (see below).

Only the most recent data point will be returned, unless additional query parameters are provided for limiting/filtering the data. Available parameters include:

    name: Filter to data with a matching name.
    start: Filter data to timestamps greater than or equal to this start timestamp.
    end: Filter data to timestamps less than or equal to this end timestamp.
    limit: The number of results to return. See "Limits" below for details.
    offset: The value to start at.

Example filtered by name: https://myfarm.farmos.net/farm/sensor/listener/xxxxxx?private_key=yyyyyy&name=temperature

Limits

The limit parameter can be used to specify how many data points should be returned. Setting limit to 0 will return the maximum number of points, which is capped at 100000 (to prevent memory exhaustion). If more data is desired, multiple requests should be made using the offset parameter. Setting limit to a value greater than the max will result in a response code of 422 Unprocessable Entity: exceeds max limit of 100000.

Data summary info

It's also possible to retrieve summary information about the data stored for a particular sensor by adding /summary to the end of the sensor's URL. This will list each unique name used in the data, along with the total count of values and the first and last timestamp for each. For example:

https://myfarm.farmos.net/farm/sensor/listener/xxxxxx/summary?private_key=yyyyyy

Example JSON response:

{
    "humid": {
        "count": 100,
        "first": "1541117473",
        "last": "1594911898"
    },
    "temp": {
        "count": 100,
        "first":"1541117473",
        "last":"1594911898"
    }
}

Allowing public API read access

Data in farmOS is private by default. A private key is required to push data to a sensor, and by default the same key is also required to pull data.

If you want to access your sensor data outside of farmOS, you should be careful not to leak your private key, because that would allow anyone to post data to your sensor.

One area where this is of particular concern is one in which you want to use a client-side language like JavaScript to pull sensor data for display on a public web page (eg: in a graph you develop yourself). Doing so runs the risk of exposing your private key, if it is included in the client-side code that is publicly visible. To allow for this use-case, you can choose to make your sensor data itself publicly available.

Listener sensors have an optional configuration setting to "Allow public API read access". Enabling this will allow data from the sensor to be queried publicly via the API endpoint without a private key.

This setting will make the data available to anyone who knows the farmOS URL and sensor public key.

If more privacy is needed, it is recommended that the sensor be kept private, and server-side API requests are used instead of client-side code.
GrovePi + Node Red

If you are looking for a DIY approach to collecting sensor data that doesn't require soldering or coding check out this guide to Collecting sensor data in farmOS using GrovePi and Node-RED on Farm Hack.
Open Pipe Kit

The Open Pipe Kit project provides a command-line script that can be used to push data to farmOS from various sensors using the Farm Sensor: Listener module. The following video demonstrates how to set it up. For more information about Open Pipe Kit, refer to: http://openpipekit.github.io

Open Pipe Kit farmOS CLI: https://github.com/openpipekit/farmos-cli
Notifications

The Farm Sensor: Listener module comes with a basic alert notification mechanism that can be configured to send an email or text message if an incoming value is above or below a given threshold.

Text messages: It is possible to send text messages by entering a special email address that corresponds to your phone number and mobile carrier.

Here are the domain names used for some of the more popular phone carriers:

    US Cellular: [number]@email.uscc.net
    Verizon: [number]@vtext.com
    Virgin: [number]@vmobl.com
    AT&T: [number]@txt.att.net
    Nextel: [number]@messaging.nextel.com
    Sprint: [number]@messaging.sprintpcs.com
    T-Mobile: [number]@tmomail.net
    republic wirelsss: [number]@text.republicwireless.com

Use the phone number of the person you are texting followed by the domain name corresponding to their carrier. For example, if you are sending a text message to a Nextel subscriber with the phone number 232-232-2323, you would enter 2322322323@messaging.nextel.com.
