# TankYou rev2
A simple 4-line linear power supply bank (9V DC) connected to the mains. 

![overview](resources/tank-you-r2.jpg)
![left](resources/tank-you-r2-left.jpg)
![front](resources/tank-you-r2-front.jpg)
![right](resources/tank-you-r2-right.jpg)
![back](resources/tank-you-r2-back.jpg)
![top](resources/tank-you-r2-top.jpg)

> [!CAUTION]
> **SAFETY WARNING**: This project involves working with **mains voltage (230V AC)**. Lethal voltages are present in the circuit. Do not attempt to build or test this device unless you are experienced in handling high-voltage electronics. Always ensure the device is unplugged before touching any internal component and that the primary section is properly insulated.


## Specifications
- power source from mains
- 2 galvanically isolated ground domains
- 4 Regulated 9V DC Lines (2 per isolated ground)
- 200mA max per line (Center-Negative polarity)
- an old-fashion external look

![package-inside](resources/tank-you-r2-inside.jpg)


## Design
The design intentionally prioritizes simplicity, availability of common components, and practical reliability over maximum electrical performance... Moreover the use of that transformer and the 4 giant and so beautiful caps.

### Schematic
![board-schematic](resources/tank-you-r2_sch.jpg)

### Circuit Analysis
The circuit is organized in three stages: transformer and rectification, filtering, and voltage regulation.

**Transformer and rectification:**<br/>
The mains transformer provides two independent secondary windings to achieve galvanic isolation between the two ground domains.
Each winding is rectified independently by bridge rectifiers, producing the following $DC$ voltage before filtering:

$V_{DC} = V_{AC} \times \sqrt{2} - 2 \times V_D$

Assuming $V_D=0.7V$:

$V_{DC}​​ = 9 \times \sqrt{2}​ - 1.4 \approx 11.33V$

**Filtering:**<br/>
Each rectified rail is filtered by bulk electrolytic capacitors ($C = 4700\mu F$).<br/>
The ripple voltage under maximum load is calculated as:

$V_{ripple} = \frac{I_{load}}{2 \times f \times C}$

At maximum combined load for each isolated domain ($I_{load} = 400mA$ total per domain, $f=50Hz$):

$V_{ripple} = \frac{0.4}{2 \times 50 \times 4700 \times 10^{-6}} \approx 0.85V_{pp}$

**Voltage Regulation:**<br/>
Four standard $7809$ linear regulators produce the four regulated $9V$ output rails. 
The input headroom available to each regulator under maximum load is:

$V_{headroom} = V_{DC} - V_{ripple} - V_{out} = 11.33 - 0.85 - 9.0 = 1.48V$

However, empirical measurements show that the transformer secondary delivers a high no-load voltage of $\approx 13VAC$, which translates to:
$V_{DC_{no\_load}}​​ = 13 \times \sqrt{2}​ - 1.4 \approx 16.9V$

Under an actual full rated load of $200mA$ per line, the transformer exhibits a typical voltage regulation drop, settling the regulator input voltage ($V_{IN}$) at $\approx 13.3V \div 13.5V$ (as documented in the Test Log). This dynamic adjustment yields a real static headroom of:
$V_{headroom\_real} = 13.3V - 9.1V = 4.2V$
Subtracting the calculated dynamic ripple ($0.85V_{pp}$), the minimum instantaneous input voltage remains at $\approx 12.45V$, which is safely above the $7809$ internal dropout threshold ($\approx 2.0V$), ensuring clean and stable regulation.


**Power dissipation:**<br/>
The thermal dissipation on each regulator decreases as the load increases due to the transformer's internal voltage drop. 
Using the real-world measured $V_{IN} \approx 13.3V$ under the maximum nominal current ($200mA$):

$P_{U_{MAX}} = (V_{IN} - V_{out}) \times I_{MAX} = (13.3 - 9.1) \times 0.2 = 0.84W$

All four regulators are mounted on a shared aluminum heatsink with an estimated worst-case thermal resistance $\Theta_{hs} \approx 7 \, K/W$. 
Assuming a high ambient temperature ($T_{amb} = 30^\circ C$), the expected stabilized temperature of the external heatsink is:

$T_{hs} = T_{amb} + (\Theta_{hs} \times (P_{U_{MAX}} \times 4)) = 30 + (7 \times 3.36) \approx 53.5^\circ C$

Accounting for the standard TO-220 junction-to-case thermal resistance ($\Theta_{jc} \approx 5 \, K/W$), the internal silicon junction temperature ($T_j$) of the hottest regulator under worst-case continuous full load is estimated as:

$T_j = T_{hs} + (\Theta_{jc} \times P_{U_{MAX}}) = 53.5 + (5 \times 0.84) \approx 60^\circ C$

**Protection:**<br/>
Each regulator is protected by an antiparallel diode connected between OUT and IN. This safe-guards the regulation stage against reverse currents from inductive or capacitive loads during power-down cycles.


## Implementation and Test

### PCB Layout
The circuit was designed with ExpressPCB.

![board-pcb-top](resources/tank-you-r2_pcb-top.jpg)
![board-pcb-bottom](resources/tank-you-r2_pcb-bottom.jpg)

### Calibration Procedure
No calibration required. Output voltages are determined by the fixed internal reference of the $7809$.

### Test Log
#### Load and Dropout Test
**Line 1**:
| Load (mA) | $V_{OUT}$ (V)  | $V_{IN} regulator$ (V) |
|:---:|:---:|:---|
|0|9.2|16.8|
|50|9.1|15.9|
|100|9.1|15.7|
|150|9.1|15.3|
|200|9.1|13.3|
|500|8.9|10.8|

**Line 2**:
| Load (mA) | $V_{OUT}$ (V)  | $V_{IN} regulator$ (V) |
|:---:|:---:|:---|
|0|9.1|16.8|
|50|9.1|15.9|
|100|9.1|15.7|
|150|9.0|14.0|
|200|9.0|13.3|
|500|8.8|10.8|

**Line 3**:
| Load (mA) | $V_{OUT}$ (V)  | $V_{IN} regulator$ (V) |
|:---:|:---:|:---|
|0|9.2|16.7|
|50|9.2|15.6|
|100|9.0|14.6|
|150|9.0|14.1|
|200|9.0|13.5|
|500|8.9|10.9|

**Line 4**:
| Load (mA) | $V_{OUT}$ (V)  | $V_{IN} regulator$ (V) |
|:---:|:---:|:---|
|0|9.2|16.7|
|50|9.2|15.6|
|100|9.2|14.6|
|150|9.1|14.1|
|200|9.1|13.5|
|500|9.0|10.9|

#### Thermal Dissipation Test
The test was conducted by drawing a continuous load of $200mA$ simultaneously on 2 lines (one per isolated dominio: line 1 and line 4) for a duration of 60 minutes, at a stable ambient temperature ($T_{amb} = 30^\circ C$):

| Elapsed (minutes) | $T_{hs} (^\circ C)$ |
|:---:|:---|
|0|30|
|15|37|
|30|39|
|45|40|
|60|40|

- **Domain 1, Line 1**: $V_{IN} \approx 13.3V$, dropping $13.3V - 9.1V = 4.2V$ across regulator U1. Power dissipation: $P = 4.2V \times 0.2A = 0.84W$.
- **Domain 2, Line 4**: $V_{IN} \approx 13.5V$, dropping $13.5V - 9.1V = 4.4V$ across regulator U4. Power dissipation: $P = 4.4V \times 0.2A = 0.88W$.

Total power dissipation delivered to the shared heatsink:
$P_{tot} = 0.84 + 0.88 = 1.72W$

Using the thermal Ohm's law, the actual thermal resistance of the external heatsink ($\Theta_{hs}$) is derived as:
$$\Theta_{hs} = \frac{\Delta T}{P_{tot}} = \frac{T_{hs} - T_{amb}}{1.72} \approx  6K/W$$


## Conclusions
**Results**: The unit ensures good regulation within the full nominal operating scale ($0 \div 200mA$), which is within standard audio/pedalboard requirements.

**Suggestions**: Add a slow-blow fuse on the transformer primary side to improve safety against internal shorts.


## About & License
**Author**: Alessandro Fraschetti (gom9000).<br/>
**Technical Notes**: The hardware design was supported by **ExpressPCB** and the custom **[expresspcb-goslib](https://github.com/gom9000/expresspcb-goslib)** libraries.<br/>
**License**: This project is licensed under the [MIT License](LICENSE).
