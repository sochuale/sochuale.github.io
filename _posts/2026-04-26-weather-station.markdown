---
layout: post
title:  "About the old weather station"
date:   2026-04-26 19:02:00 +0200
categories: home_automation 
---

How we got from zero home automation to weather station repurposing and Home Assistant during one rainy day.

### The project
I had this one on simmer for a few months and it finally hit the boiling point in one of those particularly unpleasant rainy mornings.

We have an old TFA weather station. It uses a home unit with display and 3 additional sensors. They measure temperature and humidity outside and in the closet.

I want to gather temperature and humidity data. I want to see the trends over months and I want to know how hot our server room/closet gets in summer.

#### Solution #1
This could be solved with some humidity and temperature sensors and Arduino or RPi Zero. Maybe 3D print a little case for the sensor, add a battery box...

#### Solution #2
Wait, I don't want to buy new sensors and build everything from scratch. How do I weather-proof and power the sensors? My old sensors are working fine and they're already safe to use outside. The AA batteries last forever. I just need the data from those sensors.

The specs of our weather station say the sensors use 433 MHz to broadcast the readings for the main unit to pick up. 

### Reuse, recycle
You know what else can pick up 433 MHz sensors? The old DVB-T tuner that has been lying around in a box of old hardware since DVB-T was retired in Czechia in 2020. The RTL2832U chip can receive SDR (Software Defined Radio) and there are even some cool spectrum analysis and oscilloscope projects out there if you are into that stuff.

But for this project, the SDR will suffice. Just like most ideas these days, someone already had it before me, so there's a neat [rtl_433](https://github.com/merbanan/rtl_433) GitHub project. 

If you also impulse-buy microcontrollers and other gizmos, you always have an unused piece of hardware ready to be flashed. In our case, it is an older ODROID-N2 - a great option for Home Assistant. And that's how our household got sucked into home automation as the winds outside kept howling and the cold rain drummed on our windows.

### The data pipeline
Home Assistant comes equipped with many add-ons and your quest usually is to put them together.
I feed the data from rtl_433 into the MQTT broker. Now MQTT is pretty straightforward: rtl_433 publishes the sensor readings as topics and the MQTT broker of your choice offers them to subscribers. If you live in an apartment building with 120+ units you can also subscribe to your neighbors' thermostats or the tire pressure sensors of a random car parked on the street.

My subscriber's name is Prometheus and he likes to hoard time series. Prometheus then feeds the data to Grafana that does all the pretty graphs. They both run as add-ons in Home Assistant alongside the third party add-on for rtl_433. 

![Grafana temperature data](/assets/img/grafana_temperature.png)

### Known issues
The solution is not perfect but it was easy enough to put together in a day and it used only hardware we already had at home. We've been running it for half a year and it works fine without much maintenance other than updating the Home Assistant and the add-ons.

- Our sensors were close enough to the DVB-T tuner to get picked up but the signal strength was not ideal and the tuner lacked a proper antenna. With a small antenna added later, the reliability improved significantly.

- I can't get the reading from the main unit as it does not broadcast its own temperature and humidity and only displays them.

- The sensors pick a channel and generate a random ID so the main unit or anyone else listening can differentiate between them. Replacing batteries in one of the sensors will trigger an ID change automatically. The main unit waits for its 3 sensors and if the ID disappears, it will be paired with the new one. This could potentially mess with the data collection if there was another TFA weather station nearby interfering with ours. We use the channel number to collect the data and plot the graphs but that limits the number of sensors to the number of channels. We live in a densely populated area but so far we haven't encountered any issues with interfering weather stations.

- The official Grafana add-on in Home Assistant started to get very pushy with new authentication last month which caused my data pipeline to fail. The issue itself was easily fixed with setting up proper method but the error was difficult to pin down to a specific component. Both Prometheus and Grafana insisted there was nothing wrong with the data. After some log-digging, I found a suspicious immediate rejection of connection in Prometheus logs that led me to discover the connection drop by Grafana due to authentication method mismatch.

- There are short outages in sensor readings not caused by my data pipeline, probably originating from the sensors themselves. They're short enough to not cause any issues to me.

- I still didn't get to setting up backups for the data. It's not mission critical for now though. 

### Takeaway
This was a fun and quick project that used some spare hardware and opened the world of home automation to us. I can't wait to scout the marketplace for old TFA weather stations and set up the temperature and humidity sensors at my old folks' chicken coop.

