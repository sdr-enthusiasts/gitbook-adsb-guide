---
description: >-
  If you wish to store data from readsb in a time series database such as
  InfluxDB or Prometheus, review the information below.
---

# Storing Data and Metrics in a Time Series Database

The `readsb-protobuf` container also contains InfluxData's [Telegraf](https://docs.influxdata.com/telegraf/), and can send flight data and `readsb` metrics to [InfluxDB](https://docs.influxdata.com/influxdb/) or [Prometheus](https://prometheus.io). [Telegraf](https://docs.influxdata.com/telegraf/) is not started by default. `INFLUXDBURL` must be set in order to start the built-in [Telegraf](https://docs.influxdata.com/telegraf/) instance.

## InfluxDB Options

These variables control the sending of flight data and readsb metrics to [InfluxDB](https://docs.influxdata.com/influxdb/) (via a built-in instance of [Telegraf](https://docs.influxdata.com/telegraf/)).

| Variable | Description | Default |
|----------|-------------|---------|
| `INFLUXDBURL` | The full HTTP URL for your InfluxDB instance. Required for both InfluxDB v1 and v2. | Unset |
| `INFLUXDBUSERNAME` | If using authentication, a username for your InfluxDB instance. If not using authentication, leave unset. Not required for InfluxDB v2. | Unset |
| `INFLUXDBPASSWORD` | If using authentication, a password for your InfluxDB instance. If not using authentication, leave unset. Not required for InfluxDB v2. | Unset |
| `INFLUXDB_V2` | Set to a non empty value to enable InfluxDB V2 output. | Unset |
| `INFLUXDB_V2_BUCKET` | Required if `INFLUXDB_V2` is set, bucket must already exist in your InfluxDB v2 instance. | Unset |
| `INFLUXDB_V2_ORG` | Required if `INFLUXDB_V2` is set. | Unset |
| `INFLUXDB_V2_TOKEN` | Required if `INFLUXDB_V2` is set. | Unset |
| `INFLUXDB_SKIP_AIRCRAFT` | Set to any value to skip publishing aircraft data to InfluxDB to minimize bandwidth and database size. | Unset |

## InfluxDB Schema

The database `readsb` will be created if it does not exist.

Within this database are the following measurements:

### `aircraft` Measurement

Tags and fields used for this measurement should match [Virtual Radar Server's JSON response ("the new way")](https://www.virtualradarserver.co.uk/Documentation/Formats/AircraftList.aspx).

| Tag Key | Type | Description |
|----------|------|-------------|
| `Call`   | String | The aircraft's callsign. |
| `Gnd`    | Boolean | True if the aircraft is on the ground. |
| `Icao`   | String | The ICAO of the aircraft. |
| `Mlat`   | Boolean | True if the latitude and longitude appear to have been calculated by an MLAT server and were not transmitted by the aircraft. |
| `SpdTyp` | Number | The type of speed that Spd represents. Only used with raw feeds. `0`/`missing` = ground speed, `1` = ground speed reversing, `2` = indicated air speed, `3` = true air speed. |
| `Sqk`    | Number | The squawk as a decimal number (e.g. a squawk of `7654` is passed as `7654`, not `4012`).
| `Tisb`   | Boolean | True if the last message received for the aircraft was from a TIS-B source. |
| `TrkH`   | Boolean | True if Trak is the aircraft's heading, false if it's the ground track. Default to ground track until told otherwise. |
| `VsiT`   | Number | `0` = vertical speed is barometric, `1` = vertical speed is geometric. Default to barometric until told otherwise. |
| `host`   | String | The hostname of the container. |

| Field Key | Type  | Description |
|-----------|-------|-------------|
| `Alt`     | float | The altitude in feet at standard pressure. |
| `Cmsgs`   | float | The count of messages received for the aircraft. |
| `GAlt`    | float | The altitude adjusted for local air pressure, should be roughly the height above mean sea level. |
| `InHg`    | float | The air pressure in inches of mercury that was used to calculate the AMSL altitude from the standard pressure altitude. |
| `Lat`     | float | The aircraft's latitude over the ground. |
| `Long`    | float | The aircraft's longitude over the ground. |
| `PosTime` | float | The time (at UTC in JavaScript ticks) that the position was last reported by the aircraft. |
| `Sig`     | float | The signal level for the last message received from the aircraft, as reported by the receiver. Not all receivers pass signal levels. The value's units are receiver-dependent. |
| `Spd`     | float | The ground speed in knots. |
| `TAlt`    | float | The target altitude, in feet, set on the autopilot / FMS etc. |
| `TTrk`    | float | The track or heading currently set on the aircraft's autopilot or FMS. |
| `Trak`    | float | Aircraft's track angle across the ground clockwise from 0° north. |
| `Trt`     | float | Transponder type - `0`=Unknown, `1`=Mode-S, `2`=ADS-B (unknown version), `3`=ADS-B 0, `4`=ADS-B 1, `5`=ADS-B 2. |
| `Vsi`     | float | Vertical speed in feet per minute. |

### `autogain` Measurement

| Tag Key | Type | Description |
|---------|------|-------|
| `host`  | String | The hostname of the container. |

| Field Key | Type  | Description |
|-----------|-------|-------------|
| `autogain_current_value` | float | The current gain level as set by autogain. |
| `autogain_max_value` | float | The maximum gain level as set by autogain. |
| `autogain_min_value` | float | The minimum gain level as set by autogain. |
| `autogain_pct_strong_messages_max` | float | The maximum percentage of strong messages. |
| `autogain_pct_strong_messages_min` | float | The minimum percentage of strong messages. |

### `polar_range` Measurement

| Tag Key | Type | Description |
|---------|------|-------|
| `bearing` | Number | The bearing value is between `00` and `71`. Each bearing represents 5° on the compass, with `00` as North. |
| `host`  | String | The hostname of the container. |

| Field Key | Type  | Description |
|-----------|-------|-------------|
| `range` | float | The range (in metres) at a specific bearing.

### `readsb` Measurement

| Tag Key | Type | Description |
|---------|------|-------|
| `host`  | String | The hostname of the container. |

Field keys should be as-per the `StatisticEntry` message schema from [`readsb.proto`](https://github.com/Mictronics/readsb-protobuf/blob/dev/readsb.proto).

| Field Key | Type | Description |
|---------|------|-------|
| `cpr_airborne`                  | float | Total number of airborne CPR messages received |
| `cpr_global_bad`                | float | Global positions that were rejected because they were inconsistent |
| `cpr_global_ok`                 | float | Global positions successfully derived |
| `cpr_global_range`              | float | Global positions that were rejected because they exceeded the receiver max range |
| `cpr_global_skipped`            | float | Global position attempts skipped because we did not have the right data (e.g. even/odd messages crossed a zone boundary) |
| `cpr_global_speed`              | float | Global positions that were rejected because they failed the inter-position speed check |
| `cpr_local_aircraft_relative`   | float | Local positions found relative to a previous aircraft position |
| `cpr_local_ok`                  | float | Local (relative) positions successfully found |
| `cpr_local_range`               | float | Local positions not used because they exceeded the receiver max range or fell into the ambiguous part of the receiver range |
| `cpr_local_skipped`             | float | Local (relative) positions not used because we did not have the right data |
| `cpr_local_speed`               | float | Local positions not used because they failed the inter-position speed check |
| `cpr_surface`                   | float | Total number of surface CPR messages received |
| `cpu_background`                | float | Milliseconds spent doing network I/O, processing received network messages, and periodic tasks. |
| `cpu_demod`                     | float | Milliseconds spent doing demodulation and decoding in response to data from a SDR dongle. |
| `cpu_reader`                    | float | Milliseconds spent reading sample data over USB from a SDR dongle. |
| `local_accepted`                | float | The number of valid Mode S messages accepted from a local SDR with N-bit errors corrected. |
| `local_modeac`                  | float | Number of Mode A / C messages decoded. |
| `local_modes`                   | float | Number of Mode S preambles received. This is *not* the number of valid messages! |
| `local_noise`                   | float | Calculated receiver noise floor level. |
| `local_peak_signal`             | float | Peak signal power of a successfully received message, in dBFS; always negative. |
| `local_samples_dropped`         | float | Number of sample blocks dropped before processing. A non-zero value means CPU overload. |
| `local_samples_processed`       | float | Number of sample blocks processed. |
| `local_signal`                  | float | Mean signal power of successfully received messages, in dBFS; always negative. |
| `local_strong_signals`          | float | Number of messages received that had a signal power above -3 dBFS. |
| `local_unknown_icao`            | float | Number of Mode S messages which looked like they might be valid but we didn't recognize the ICAO address and it was one of the message types where we can't be sure it's valid in this case. |
| `max_distance_in_metres`        | float | Maximum range in metres |
| `max_distance_in_nautical_miles`| float | Maximum range in nautical miles |
| `messages`                      | float | Total number of messages accepted by readsb from any source |
| `remote_accepted`               | float | Number of valid Mode S messages accepted over the network with N-bit errors corrected. |
| `remote_modeac`                 | float | Number of Mode A / C messages received. |
| `remote_modes`                  | float | Number of Mode S messages received. |
| `tracks_mlat_position`          | float | Tracks consisting of a position derived from MLAT |
| `tracks_new`                    | float | Total tracks (aircraft) created. Each track represents a unique aircraft and persists for up to 5 minutes. |
| `tracks_single_message`         | float | Tracks consisting of only a single message. These are usually due to message decoding errors that produce a bad aircraft address. |
| `tracks_with_position`          | float | Tracks consisting of a position. |

## Prometheus Options

These variables control exposing flight data and `readsb` metrics to [Prometheus](https://prometheus.io) (via a built-in instance of [Telegraf](https://docs.influxdata.com/telegraf/)).

| Variable | Description | Default |
|----------|-------------|---------|
| `PROMETHEUS_ENABLE` | Set to any string to enable Prometheus support | Unset |
| `PROMETHEUSPORT` | The port that the Prometheus client will listen on | `9273` |
| `PROMETHEUSPATH` | The path that the Prometheus client will publish metrics on | `/metrics` |
