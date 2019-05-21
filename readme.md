# GoPro Telemetry

**Work in progress. Don't rely on it for important stuff yet.**

Reads telemetry from the GPMF track in GoPro cameras (Hero5 and later).

Created for the [GoPro Telemetry Extractor](https://tailorandwayne.com/gopro-telemetry-extractor/).

Here's a [playlist with cool uses of the GoPro metadata ](https://www.youtube.com/watch?v=V4eJDQik-so&list=PLgoeWSWqXedK_TbrZXg7L926Kzb-g_CXz).

Accepts an object with binary data and timing data. Returns a JavaScript object with a key for each device that was found. See **samples/example.js** for a basic implementation.

You must extract the raw GMPF data from the video file first. You can do so with [gpmf-extract](https://github.com/JuanIrache/gpmf-extract).

**gpmf-extract** will provide you with an object ready to import. It contains:

- **rawData** (buffer) The GPMF track of the video file.
- **timing** (object) Provides timing information such as starting time, framerate, payload duration... as extracted from [gpmf-extract](https://github.com/JuanIrache/gpmf-extract).

Install:

```shell
$ npm i gopro-telemetry
```

Use:

```js
const goproTelemetry = require('gopro-telemetry');
const telemetry = goproTelemetry(input, options); //Get your input with gpmf-extract
```

## Options

Some options may be incompatible with others.

- **debug** (boolean) Outputs some feedback. Default: _false_
- **tolerant** (boolean) Returns data even if format does not match expectations. Default: _false_
- **deviceList** (boolean) Returns an object with only the ids and names of found devices. **Disables the following options**. Default: _false_
- **raw** (boolean) Returns the data as close to raw as possible. No matrix transformations, no scaling. **Disables the following options**. Default: _false_
- **device** (array of numbers) Filters the results by device id. Default: _null_
- **stream** (array of string) Filters the results by device stream (often a sensor) name. You can find information on what many sensors are called [here](https://github.com/gopro/gpmf-parser#where-to-find-gpmf-data). Default: _null_
- **repeatSticky** (boolean) Puts the sticky values in every sample and deletes the 'sticky' object. Default: _false_
- **repeatHeaders** (boolean) Instead of a 'values' array, the samples will be return under their keys, based on the available name and units. Default: _false_
- **timeIn** (string) By default the code uses MP4 time for _cts_ (milliseconds since first frame) and GPS time for _date_ (full date and time). Specify one (**MP4** or **GPS**) in order to ignore the other. Default: _null_ (imports both)
- **timeOut** (string) By default the code exports both _cts_ and _date_. Specify one (**cts** or **date**) in order to ignore the other. Default: _null_ (exports both)
- **groupTimes** (number/string) Group samples by units of time (milliseconds) for example, if you want one sample per second, pass it 1000. It also accepts the string **frames** to match the output to the video frame rate. Default: _null_

Example:

```js
const telemetry = goproTelemetry(rawData, { stream: ['ACCL'], repeatSticky: true });
```

This slightly more comprehensive example includes the data extraction step with [gpmf-extract](https://github.com/JuanIrache/gpmf-extract).

```js
const gpmfExtract = require('gpmf-extract');
const goproTelemetry = require(`gopro-telemetry`);
const fs = require('fs');

const file = fs.readFileSync('path_to_your_file.mp4');

gpmfExtract(file)
  .then(extracted => {
    let telemetry = goproTelemetry(extracted);
    fs.writeFileSync('output_path.json', JSON.stringify(telemetry));
    console.log('Telemetry saved as JSON');
  })
  .catch(error => console.log(error));
```

**timing** object structure:

```
{ frameDuration: 0.03336666666666667,
  start: 2017-04-17T19:27:57.000Z,//Date object
  samples:
   [ { cts: 0, duration: 1001 },//Starting point and duration in milliseconds
     { cts: 1001, duration: 1001 },
     { cts: 2002, duration: 1001 },
     { cts: 3003, duration: 1001 },
     { cts: 4004, duration: 1001 } ] }
```

## Output

The output with the default options looks like this:

```
{ deviceId : {
    data about the device : values,
    streams : {
      stream_key : {
        data about the samples : values,
        samples : [
          {
            cts : time from start,
            date : time and date,
            value : sample
            sticky : {
              name : value
            }
          },
          {
            cts : time from start,
            date : time and date,
            value : sample
          }
        ]
      }
    }
  }
}
```

Sticky values apply to all successive samples. You can export them to the outer object of all samples with the **repeatSticky** option.

## Available data

Depending on the camera, model, settings and accessories, these are some of the available data:

- GPS location (deg, m)
- GPS speed (m/s)
- Accelerometer (m/s²)
- Gyroscope (rad/s)
- ISO
- Shutter Speed (s)
- Timestamps (µs)
- Magnetometer (µT)
- Face detection
- Highlights (manual and computed)
- White balance
- Luma
- Hue
- Image uniformity
- Scene classifier

This project is possible thanks to the [gpmf-parser documentation](https://github.com/gopro/gpmf-parser), open sourced by GoPro.

## More creative coding

If you liked this you might like other [creative coding projects](https://tailorandwayne.com/coding-projects/).

## To-Do

- Interpret data
  - Enable grouping packets per time unit / frame
  - Automated test interpretation
- Fix typos
- Review console.log/error usage
- Create additional package/option for converting the data to other formats
- Remove Work-in-progress warning

## Maybe To-Do

- Do something with TICK and TOCK?
- Take potential nested arrays into account f[8]? Never found one to test
