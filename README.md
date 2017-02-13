# sony_ep30_bluetooth
hacking a Sony EP30 compact stereo combo to support bluetooth

## parts
* Sony EP30 compact stereo
* KRC-86B bluetooth module
* blue LED (included)
* 47uF cap (included)
* LM7805
* 0.33uF foil cap (or ceramic)
* 0.1uF foil cap (or ceramic)
* some piece of prototyping board

## resources
* Sony EP30 service manual http://diagramas.diagramasde.com/audio/HCD-EP30%20EP40%20sm.pdf
* https://hackaday.io/project/10794-portable-stereo-speaker
* http://www.ebay.de/itm/401223081706

## goals
* bluetooth audio playing as long as bluetooth is connected
* else: tape audio passthrough
* bluetooth module is powered only powered if the system is powered (using internal power supply)

## build log
### stage 1: audio test with external supply voltage
* supply via 5V breadboard supply + 9V wall wart
* simply soldering the L/R/GND to PL/PR/GND on the back side of the tape connector (labeled *CASSETE BOARD/CN201*)
* **audio does only play while the hardware tape play button is pressed**
* *caveats:* tape motor is running

### stage 2: using the CD audio input
* soldering the L/R/GND to back side of the CD connector (labeled *CN203*)
* **could not get the audio to play at all**
* I think the output is muted (*CN302/MUTE*) as long as ne CD data is read

### stage 3: back to tape
* soldering the L/R/GND to back side of the tape connector (labeled *CASSETE BOARD/CN201*)
* unplugging the tape play switch and tape motor (labeled *CN202*)
* shorting the *S201* un-muted the output, bluetooth audio is playing

### stage 4: power supply
* LM7805 constant voltage supply circuit is shown in the datasheet
* input voltage is +9V, soldered to the back side of the *CN202* connector (V+)
* ground soldered to *CN202* (M-)
* shorted *S201*
* +5V output is connected to KRC-86B VIN
* GND is connected to KRC-86B GND
* **bt-audio is working but there is noise**
* put everything back together

### stage 5: audio troubleshooting
resources:
* http://onlinetonegenerator.com/
* http://mathworld.wolfram.com/FourierSeriesTriangleWave.html

there are two types of noise present:
* a solid lower frequency humming, by comparison it sound like 50Hz triangle, according to the fourier transform of the triangle wave it contains 50Hz*(2n+1)
* high pitched noise if bluetooth is in pairing mode

solutions:
* **removing KRC-86B AGND -> decreasing low-frequency humming volume, but still present without sound playing**
* the system uses a common ground for both audio and V+, so disconnecting AGND removed a grounding loop
* moving the grounding to the central GND point of the systems power board could prove beneficial in a *star grounding* ansatz

**final thoughts for this stage of the project:**
* the high-frequency noise could be induced by the unshielded bluetooth circuit, but since it is all on one board, putting the circuit into a shielding box would not work
* I would like to look at the L/R and especially GND line with an oscilloscope and maybe spectrum analyzer (kinda overkill here) to perform a frequency analysis
* regaining tape functionality could be achieved by breaking the solder bridge (made by me) on *S201* and plugging the connector *CN202* back in
* closing *SN201* sets COTR on *CN201* high (+9V) (traces to the muting switch control), so VIN and GND should be moved to *CN201* and a transistor could pull COTR to +9V if EN on KRC-86B is high (+3.3V) (high if bluetooth device is paired). No damage should be done since the IC seems to be protected by a diode from the COTR signal
