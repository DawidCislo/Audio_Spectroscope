# Audio_Spectroscope
![](https://github.com/DawidCislo/Audio_Spectroscope/blob/main/GIF_Preview.gif)

This is an Audio Spectroscope made without using any kind of programmable devices. It uses over 120 ICs such as operational amplifiers, comparators and logic gates to create the effect you can see in the preview above.

# Theory of operation
The device consists of 11 PCBs, one backplane and 10 display modules. You can find the KiCad project files for the backplane in the 'ASP_Backplane' folder, and files for the display modules in the 'ASP_DIsplay' folder. On the backplane there are 2 3.5mm TRS sockets. Both of them are connected together, and the user can plug the source to one of the sockets, and headphones or speakers to the other, so there's no need to use a splitter. Next using a set of jumpers (JP1, JP2, JP3) the user can use either the left, right or both channels as the input for the spectroscope. Jumpers feed into a preamp with user-adjustable gain (U4A), so both quitet and loud listeners can see the spectroscope work. The series shottky diode (D4) protects the preamp from voltage spikes coming from the audio input and can be omitted.

![](https://github.com/DawidCislo/Audio_Spectroscope/blob/main/Docs/Prev_Backplane_Audio_In.png)

The preamp feeds into 10 active band-pass filters that separate the audio signals into different bands. Those bands are: 32Hz, 63Hz, 125Hz, 250Hz, 500Hz, 1kHz, 2kHz, 4kHz, 8kHz and 16kHz. Each band is connected to a separate display module connector.

![](https://github.com/DawidCislo/Audio_Spectroscope/blob/main/Docs/Prev_Backplane_Filters.png)

The backplane also has a power connector (5.5/2.1mm DC jack, center positive), for use with an external 12V 1A power supply, with a switch and some basic overvoltage protection. You can save on component cost by replacing the power switch (SW1) with a jumper, so that the device will always be on, and you can omit the TVS diode (D1) and replace both the fuse (F1) and reverse-circuit protection diode (D3) with a short - this will reduce resistance to voltage spikes coming from the external power supply.

![](https://github.com/DawidCislo/Audio_Spectroscope/blob/main/Docs/Prev_Backplane_Power_In.png)

Device uses an LM2576T for generating the 5V rail, the circuit has a voltage divider with resistor values calculated for an adjustable version of LM2576T, but if a fixed 5V version of this IC is available, simply omit R13 during assembly and replace R12 with a short.

![](https://github.com/DawidCislo/Audio_Spectroscope/blob/main/Docs/Prev_Backplane_5V_Supply.png)

Backplane also has a -10V source built using an MC34063 multi-purpose controller. Due to low current draw on the -10V rail, internal switching transistor is sufficient, simplifying the design.

![](https://github.com/DawidCislo/Audio_Spectroscope/blob/main/Docs/Prev_Backplane_-10V_Supply.png)

Each display module has two peak detectors connected in series. The first detector made up of U3A, D1 and U3B has a smaller hold capacitor C12 and it's quickly discharged by R34. Output of this peak detector will show the current amplitude of a filtered audio signal. Next peak detector (U3C, D2 U3D) has a larger hold capacitor and it does not have a discharge resistor. Since it's connected to U3D and D2 it will still discharge, because of non-zero leakage currents of those components. Output of this slow peak detector will show the maximum amplitude of the audio signal and hold it there for a short while.

![](https://github.com/DawidCislo/Audio_Spectroscope/blob/main/Docs/Prev_Display_Detectors.png)

Both the fast and slow peak detector outputs are connected to 12 comparators each. Reference levels for the comparators are set by a long voltage divider in the middle. If a voltage on the input line is higher than the reference voltage set by the resistors, a comparator pulls the output low (since it has an open-collector output). In effect, the higher the voltage on the input line, the more comparators are switched off, and the more of their outputs have a logic low. On the fast side the comparators connect to logic NOT gates through some simple Resistor-Diode level shifter. On the slow side the comparators feed through voltage level shifters into Ex-OR gates. Each Ex-Or gate is connected to both it's own comparator, and the next comparator. If both of the comparator outputs are the same, the gate's output is low. Only in a situation, when the current level is different from the next one, the gate has a logic 1 on its output. As a result, for example, the fast side outputs '000001111111' while the slow side wil only output '000001000000'. 

![](https://github.com/DawidCislo/Audio_Spectroscope/blob/main/Docs/Prev_Display_ADC.png)

The LEDs are driven by a pair of transistors each (Q2, Q3, Q4 ...). One transistor is controlled by the slow side, while the other one is controlled by the fast side. Transistors are connected in such a way, that either one can independently turn the LED on. Current of the LEDs is set by the series resistors (R8, R9, R10 ...), increasing the value of these resistors decreases the brightness of the LED, and reducing the value increases it. Different LEDs from different manufacturers might need adjustment of these resistor values to achieve a matching brightness for different colors. During the assembly of the first prototype I bought new red and green LEDs, but used old yellow LEDs that I had in stock. Since the old LEDs were much dimmer than the new ones, I used a lower value of resistance for yellow LEDs. Results may vary for your LEDs and it's much better to check them before soldering all 10 display modules.

![](https://github.com/DawidCislo/Audio_Spectroscope/blob/main/Docs/Prev_Display_LEDs.png)


#  Assembly tips and troubleshooting
>[!CAUTION]
>This device is ***VERY CHALLENGING TO ASSEMBLE*** There is a lot of small and hard to solder components (0603, TSSOP with 0.65mm pin pitch 4x0603 resistor networks, SOT-363s) packed very close together. On top of that there's over 1200 components to solder in total, and it will take very long time to assemble.
I recommend testing every PCB before connecting them together:
- Display modules: Connect all the supply rails on the angled connector to a corresponding voltage source, preferably use a multi-rail adjustable power supply. Next, connect the input of the display module to a potentiometer, generating a voltage between 0V and 5V. See if all the LEDs light up. Next rapidly turn the potentiometer down and see if all the LEDs light up as the slow peak detector discharges. If you see LEDs that are permamently off, check the series resistors and dual transistors next to the LEDs. If whole 4x or 6x sections refuse to light up or stay always on, check the ICs. If single LEDs stay on or light up in the wrong order, check the resistor networks. If the whole module refuses to light up, check the peak detector.
- Backplane: Check if there is no short on any power rails by sticking two pieces of wire into any of the display module connector and using a multimeter in a continuity mode. If there are no shorts on any of the rails, connect the external power supply, and then using the same 2 wires and a multimeter, now in voltage mode, check if all the supply rails have a correct voltage. Note: having a slightly lower voltage on both the 12V and -10V is still fine and will not prevent the device from wirking correctly.
Only after each PCB is thoroughly tested, you can connect them all together and see if the device really works. Display modules are packed very close together, so make sure that the LEDs don't short to the neighbouring modules. Now connect the audio cable and use an online tone generator ([like this one](https://www.szynalski.com/tone-generator/)) to test if the band-pass filters are working correctly.

![](https://github.com/DawidCislo/Audio_Spectroscope/blob/main/Docs/Prev_Assy.jpg)

If everything is working as it should, you can go ahead and assemble it together. I used a set of long M3 screws to tie all the display modules together and used a [TEKAL 31.29](https://www.tme.eu/pl/details/tekal31.29/obudowy-z-panelem/teko/tekal-31-29/) case. I've used thin strips of gray foil to stop light from bleeding between LEDs. In this preview you can see the first prototype, which had several major errors - they have been fixed and the files shared in this repository should be all good. On the animated preview and here, only 9 display modules are fitted - last module is broken and I can't figure out what exactly is wrong with it.
> [!NOTE]
> All those ICs packed so close together get pretty hot, and the hotter the circuit gets, the larger leakage currents are, and the faster the 'slow' peak detector falls. Drill a few ventillation holes or use a thermal pad to help dissipate the heat.

## Other information 
The project files were last edited in KiCad 7.0.9 and can be opened by any version that's newer than that.

Symbols and footprints used to create this project are part of my library, you can get it and use it for your projects here: [alternatekicadlibrary.com](https://alternatekicadlibrary.com/).

All the content within this repository is dedicated to the public domain under the [CC0 1.0 Universal (CC0 1.0) Public Domain Dedication](https://creativecommons.org/publicdomain/zero/1.0/).
