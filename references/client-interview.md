#2023.03.31

Client specified that they wanted to accurately determine battery charge and allow for charging up to a user specified number of batteries for each type. They described a modular system that incorporates a user determined amount of modules for each battery type (AA, AAA, drill batteries, flashlight batteries). This storage solution is meant to help users cycle through their batteries in storage in an order that will improve battery life.

This project will include two major components, the microcontroller unit and 3D printed housing and the code base for user interface. Both will be open source. The general idea is to set a DIY approach along with an out-of-the-box solution.

What makes this particular project difficult is the number of and variety of voltages that need to be monitored along with the potential for higher amperages. A potential solution to this problem is using a MOSFET.

Some discussion was had about how to obtain current charge information for batteries. Should one voltage sensor be used for the system? Or should one voltage sensor be used for each battery? It was decided that one voltage sensor should be used for the entire system. 

Should users input which modules have batteries and which ones don't? Is this something that can automatically be checked for? Yes, batteries with no charge, even dead ones, will pull on the existing electronics changing the voltage state. No, users should not input which modules are active. 

Dock: 3D printed structure that actually houses user batteries
Battery charger: this is something that the user will provide that matches the battery type that they are trying to charge
"Dummy battery": this is 3D printed and will go into the user provided battery charger with electrode connections to complete a circuit
Bus line: this is the negative for all batteries in the dock
Individual lines: these will connect to each battery in the dock

Ultimately a series of PINs will need to be activated and information run through certain sensors on the microcontroller.

There will need to be a way for users to input information, in this case thresholds. Discussion about percentages or voltages. Client is insistent about having voltages be the input.
For AA, the total voltage is 1.5V, but the user wants to store between 1.0 and 1.2, those would be the lower and upper thresholds respectively.
1.2-1.5 green on led
1.0-1.2 blue on led
<= 1.0 red on led
This needs to happen for every battery.

Refresh interval should be no shorter than 6 hours. Have to allow batteries to charge between reading cycles. Gives code a start condition. Also informs that time keeping is going to be important in the code, it may need wait or sleep periods.

Client wants microcontroller to be flashable

V = IR
Power = work done, some relationship between V and I

Whether or not to include a hard reset button or start button or something.

#2023.04.01
https://predictabledesigns.com/how-to-switch-large-loads-with-a-microcontroller-using-transistors/
https://electronics.stackexchange.com/questions/546709/understanding-mosfet-characteristics
Client tools max chargers are about 24 V (21 actual) and 13.5 Amps. So need a transistor that can handle ~ 40 Volts and 15+ Amps.
https://www.ti.com/lit/ds/symlink/csd17578q5a.pdf?HQS=dis-dk-null-digikeymode-dsf-pf-null-wwe&ts=1680368152329&ref_url=https%253A%252F%252Fwww.ti.com%252Fgeneral%252Fdocs%252Fsuppproductinfo.tsp%253FdistId%253D10%2526gotoUrl%253Dhttps%253A%252F%252Fwww.ti.com%252Flit%252Fgpn%252Fcsd17578q5a

For 18650s (flashlight), example user settings: Keep between 3.64 Volts and 3.71 Volts. Keep above 10C. Keep x number of batteries fully charged.
https://batterybro.com/blogs/18650-wholesale-battery-reviews/77975750-how-to-store-18650-batteries-safely
18650 chargers need maybe a 5 Amp 5-10 Volt rated MOSFET.
https://www.ti.com/lit/ds/symlink/csd22206w.pdf?HQS=dis-dk-null-digikeymode-dsf-pf-null-wwe&ts=1680369632870&ref_url=https%253A%252F%252Fwww.ti.com%252Fgeneral%252Fdocs%252Fsuppproductinfo.tsp%253FdistId%253D10%2526gotoUrl%253Dhttps%253A%252F%252Fwww.ti.com%252Flit%252Fgpn%252Fcsd22206w
Wonder if the bigger one [MOSFET] will work with the smaller voltages/loads, so only one version of this is needed?
Increasing modularity?
For AA's... 8 Volts max, 3 Amps
https://www.vishay.com/docs/67999/si8802db.pdf

#2023.04.03
Sent to client: https://link.medium.com/ehENX6hQHyb
My observation from this link is that Micropython will be necessary
From client: Wi-Fi means sending emails when something happens or weekly reports
From client: https://www.youtube.com/watch?v=bTYQ_Jrpz6Y&ab_channel=educ8s.tv
From client: "I like the idea of broadcasting a wifi signal connecting to my phone and setting the user inputs that way on a self-hosted 'website'".
My observation from comment: Web application for user to input information
From client: "vs connecting to existing wifi and figuring out how to do all of that securely \br either case wifi is helpful. I know nothing how to implement any of that"
From client: https://www.digikey.com/en/products/detail/raspberry-pi/SC0918/16608263
https://www.youtube.com/watch?v=dFDGtlSi9Eg&ab_channel=JeffGeerling

Pico has 23 usable gpio pins (3 more are weird). 
The 3 weird ones can measure temp or voltages!
The 23 other ones can control transistors/mosfets/relays/whatever.
So, to be able to charge 24 batteries max:
1 weird for voltage check
22 normal for battery relays
1 normal for main relay (cant use a weird one because of power leak)
2 final weirds for last 2 battery relays.

Cant have a separate temp sensor (pico has a temp sensor, but will read higher than ambient, have to measure/adjust) 
Heres a voltage tester tutorial. Anything needing above 3.3volts tested needs other stuff to get it within testable range.
https://microcontrollerslab.com/raspberry-pi-pico-adc-tutorial/

Heres basically what we need: Voltage tester. Have a 0V level, low battery level, good battery levels. Driving status LED's depending on what levels detected.

Uses a voltage divider to step down battery voltage to the 0-3.3V range usable by the Pico. This case is 2 resisters chosen to step 11.2V down to 3.2. 

https://github.com/aboudou/picheckvoltage

output of the pins is 3.3V, 8mA by default. Its a 3.3V device,s o that wont change. But we can manually set the output to 16mA. Suggest just leaving it at 8mA.

Transistor is current controlled, mosftet is voltage (safer for gpio).

11 minute mark is really cool. A single chip can control 8 different channels. The one specifically shown is maxed out at 500mA per circuit.

https://www.youtube.com/watch?v=6KoJ_D2ashI&ab_channel=Kitflix
https://electronics.stackexchange.com/questions/367274/what-is-drive-voltage-for-a-mosfet

Really good animation, explaining why you need two transistors to use a mosfet to act as a switch. 2nd link gives model numbers for using a 60V/30A mosfet as a switch.
https://raspberrypi.stackexchange.com/questions/67166/raspberry-pi-mosfet-npn-transistor-24v-solenoid-circuit-gets-hot

https://quadmeup.com/raspberry-pi-mosfet-high-power-switch/