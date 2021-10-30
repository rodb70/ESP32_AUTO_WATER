# ESP32_AUTO_WATER

An ESP32 module together with a capacitive soil moisture sensor and DS3231 Real-time Clock (RTC) is used to automatically water a plant bed using a pump placed in a water storage container. 

Power for the pump comes from a bank of supercapacitors charged by a solar panel. The solar panel also charges a battery that supplies power to the electronics.

The system is self-contained. No mains power supply or connection to a water faucet is required, so this can be used safely un-attended for long periods (several weeks), until the water storage runs out.

In normal watering mode, the ESP32 is woken up from deep-sleep mode once a day by the DS3231 RTC at a scheduled time. It then checks the soil moisture level and if required, turns on the water pump. It then logs the calendar date and time, moisture sensor reading, power supply voltages and watering duration as entries in a Google Docs spreadsheet document. 

If you reset the ESP32 module and then press a configuration button at an audio prompt, the system is configured as a standalone WiFi Access Point (AP). A web server page running on this AP can then be used to configure the scheduled watering time, moisture threshold for watering, RTC clock time, internet access SSID and password credentials (for the Google Docs spreadsheet update). 

You can also update the firmware via this WiFi AP, as the watering system may not be conveniently located for serial port cable-based programming.


## Power supply

* Solar panel<br>
A 20V solar panel is used to charge a lead acid battery via a CC-CV module. It is also used to charge up a bank of supercapacitors, as it's possible that at the scheduled watering time, it may be cloudy enough that the water pump will not run off the solar panel.

* Lead acid battery 6V 1.3Ah<br>
This is used to provide power for the ESP32 module via a 3.3V regulator. You could use a 3.7V li-ion battery instead with a Li-ion MPPT solar charger module.<br>
I used an HT7333 LDO 3.3V regulator because it draws very little quiescent current (< 5uA) and can handle higher input voltages. <br>
When the ESP32 module is active the current draw is approximately 25mA on average. <br>
When the ESP32 module is in deep sleep, the additional circuits drawing current are :
- HT7333 regulator quiescent current
- DS3231 RTC
- The resistors providing the sensed voltages to the ESP32 ADC. I used 500K potentiometers to minimize the current draw, with a 100nF capacitor on the ADC pins to minimize noise due to the high source impedance.

* Supercapacitor bank<br>
I used eight 10F 2.7V supercapacitors in series to provide an energy storage bank to power the 12V water pump.<br>
Even on a moderately cloudy day, the capacitor bank slowly charges up to approximately 18V.<br>
I used a series 1N4007 diode from the solar panel so that the capacitor bank does not discharge back through the panel or the CC-CV module. Once charged to ~18V there's enough juice in the supercapacitor bank to run the 12V water pump for 30+ seconds.

### DS3231 RTC 
This provides a daily alarm at the scheduled time to wake up the ESP32 from deep sleep. A 4.7uF capacitor in series between the DS3231 INT/SQW pin and the ESP32 EN pin generates a reset pulse for ESP32 wake-up.

For the ESP32 EN pin, I used a 2K2 resistor pullup to VCC and a 1uF ceramic cap to ground. This is enough to avoid ESP32 reset issues while still allowing the DS3231 reset pulse to work.

A CR2032 coin cell battery provides backup for the DS3231 RTC. 

### Capacitive water sensor
[Ensure you have a working capacitive sensor module](https://www.youtube.com/watch?v=IGP38bz-K48). The version I have uses a 555 timer IC marked "NE555 20M". 

I sealed the electronics back-end of the sensor board with silicone caulk and a heatshrink tube to prevent any corrosion of the electronics. A 1-meter shielded cable provides ground, 3.3V and sensor output interface to the ESP32.

It's possible for the top soil layer to dry out while the roots are still in damp soil. So the sensor is placed halfway down the side of the plant pot, 

### CC CV battery charger module
I used an LM2596 module to charge the 6V 1.3Ah lead acid battery. I set open circuit voltage (CV) to 7.2V, and short-circuit current (CC) to 0.25A.