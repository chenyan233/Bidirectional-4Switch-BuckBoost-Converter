# 100 W Bidirectional Four-Switch Buck-Boost Converter Design

[中文](README.md) | **English**

<p align="center">
  <img src="3D%E7%AC%AC%E4%B8%80%E5%BC%A0_3DHero.png" alt="3D view of the converter PCB" width="900">
</p>

<p align="center">
  <img src="PCB%E9%A1%B6%E9%9D%A2_PCBTop.png" alt="PCB top side" width="49%">
  <img src="PCB%E5%BA%95%E9%9D%A2_PCBBottom.png" alt="PCB bottom side" width="49%">
</p>

## Project Status

| Item | Current status | Notes |
|---|---|---|
| PSIM simulation | Preliminary validation completed | Covers bidirectional buck and boost operation, the transition region, and representative CCM and DCM conditions |
| Main power PCB and controller adapter PCB | Bare boards fabricated | Component assembly, power-up, and full-power testing are still pending |
| Efficiency, ripple, and protection | Hardware measurements pending | Values in this report are design targets or simulation results, not measured hardware performance |

This report documents the design rationale, control approach, and preliminary validation of the first hardware revision. Efficiency, ripple, temperature rise, transient response, and protection measurements will be added as hardware testing progresses.

## 1 Project Background

This project grew out of my microgrid project.

In that project, I designed a flywheel energy storage system (FESS) and a battery energy storage system (BESS), both connected to the inverter's common DC bus. Bidirectional energy exchange with the grid allows the storage systems to charge and discharge, supporting peak shaving, power balancing, and power quality regulation.

While designing the BESS, I needed a bidirectional DC-DC converter between the storage unit and the inverter DC bus. I chose the four-switch bidirectional buck-boost topology to turn that system-level requirement into hardware I could build and test. That was the starting point for this project.

### 1.1 Power Rating and Hardware Scope

My microgrid design and simulation work is mainly carried out in MATLAB/Simulink. In the original model, the inverter DC bus reaches approximately 1200 V and the BESS power rating is approximately 50 kW.

Building a converter directly at that voltage and power level would be impractical for a personal hardware project, considering safety, cost, component selection, and the available laboratory facilities. I therefore retained the bidirectional buck-boost topology and scaled down the voltage and power for the hardware implementation.

The lower rating still leaves substantial engineering work. The design retains bidirectional power flow, buck and boost operation, continuous conduction mode (CCM) at rated load, and discontinuous conduction mode (DCM) at light load.

The controller uses an outer voltage loop and an inner current loop. The hardware work covers MOSFETs, the inductor, capacitors, gate drivers, and bidirectional current sensing, as well as overcurrent and overvoltage protection. Switching ringing and parasitic effects also need to be addressed.

For the PCB, I focused on the placement and return paths of the high-current power loops, gate-drive loops, and Kelvin sensing connections. Although the converter is rated at only around 100 W, the project retains most of the power electronics design process: topology, control, component selection, protection, PCB layout, simulation, and subsequent hardware testing.

## 2 Design Specifications

| Parameter | Design value |
|---|---|
| Topology | Four-switch synchronous, non-isolated, bidirectional buck-boost |
| Rated power | 100 W |
| Port A voltage | 18–30 V |
| Port B voltage | 24 V nominal control operating point |
| Power flow | Bidirectional |
| Switching frequency | 100 kHz |
| Design current ripple ratio | $r = 0.4$ |
| Rated-load operation | CCM |
| Light-load operation | DCM; control strategy still being refined |
| Target efficiency | 98.5%; design target, pending measurement |
| Output voltage ripple target | ≤0.5% for the first revision; 0.1% as a stretch target |

The 24 V value at port B is the nominal operating point used for the current control design and simulations. It is not an inherent restriction of the topology. Either port can act as the input or output over a voltage range, subject to component ratings, protection thresholds, and control limits. Holding port B at a nominal 24 V represents its use with a 24 V battery or a low-voltage DC bus.

## 3 Power Stage Design

### 3.1 Main Inductor

The initial inductor calculation uses the current ripple ratio:

$$
r = \frac{\Delta I_{L}}{I_{L,avg}} = 0.4
$$

Because either port can serve as the input or output, I checked four boundary conditions: 18 V → 24 V, 30 V → 24 V, 24 V → 18 V, and 24 V → 30 V.

The 100 kHz switching frequency is a compromise between magnetic component size, switching losses, control bandwidth, and EMI. It gives a switching period of 10 µs. With a design ripple of 40% of average inductor current, I calculated the inductor volt-seconds and required inductance for each boundary condition.

For 18 V → 24 V, the volt-second product is approximately 45 V·µs, giving a theoretical inductance of 20.25 µH. For 30 V → 24 V, it is approximately 48 V·µs, giving 28.8 µH. The reverse cases, 24 V → 18 V and 24 V → 30 V, give 20.25 µH and 28.8 µH respectively. The highest theoretical minimum among these cases is therefore approximately 28.8 µH.

For example, in the 18 V → 24 V boost case, the ideal duty cycle is:

$$
D = 1 - \frac{18}{24} = 0.25
$$

The inductor volt-second product over one cycle is:

$$
18 \times 0.25 \times 10 = 45\text{~V} \cdot \mu s
$$

At 100 W, the average current on the 18 V side is approximately:

$$
I_{L} = \frac{100}{18} \approx 5.56\text{~}A
$$

With a 40% ripple target:

$$
\Delta I_{L} \approx 0.4 \times 5.56 = 2.22\text{~}A
$$

The required inductance is approximately:

$$
L = \frac{45}{2.22} \approx 20.25\text{~}\mu H
$$

Rather than selecting a value right at the highest theoretical minimum of 28.8 µH, I chose a commercially available 33 µH inductor to allow for manufacturing tolerance and reductions in effective inductance with temperature and DC bias. Saturation current, temperature-rise current rating, DCR, and core losses must also be checked.

#### 3.1.1 Ripple Check with 33 µH

At 45 V·µs:

$$
\Delta I_{L} = \frac{45}{33} \approx 1.36\text{~A}
$$

At 48 V·µs:

$$
\Delta I_{L} = \frac{48}{33} \approx 1.45\text{~A}
$$

The 33 µH selection therefore provides some margin over the theoretical minimum. Final selection also depends on DCR, effective inductance under DC bias, saturation current, RMS current capability, core losses, and temperature rise at 100 kHz. It is a balance between current capability, losses, temperature, and size, rather than nominal inductance alone.

The intended minimum requirements are:

| Parameter | Target |
|---|---|
| Nominal inductance | 33 µH |
| Effective inductance at high current | Preferably around 28.8 µH or higher |
| Saturation current | ≥10 A |
| Temperature-rise current rating | ≥8 A |
| DCR | <15 mΩ |
| Construction | Shielded inductor |

Low DCR reduces copper losses:

$$
P_{Cu} = I_{L,RMS}^{2}R_{DCR}
$$

A shielded inductor also helps reduce magnetic interference with current sensing and control signals on the PCB.

### 3.2 Capacitors

The initial output capacitance calculation uses 18 V → 24 V boost operation. Among the four nominal boundary conditions, this places the highest capacitance requirement on the 24 V output side.

The boost duty cycle is:

$$
D = 1 - \frac{V_{in}}{V_{o}}
$$

Substituting the voltages:

$$
D = 1 - \frac{18}{24} = 0.25
$$

At 100 W output power:

$$
I_{o} = \frac{100}{24} \approx 4.17\text{~A}
$$

Ignoring ESR and ESL, the preliminary boost output capacitance calculation is:

$$
C_{\min} = \frac{I_{o}D}{f_{s}\Delta V_{o}}
$$

Using 0.1% of the 24 V output as an idealized stretch target:

$$
\Delta V_{o} = 24 \times 0.1\% = 0.024\text{~V}
$$

This gives:

$$
C_{\min} = \frac{4.17 \times 0.25}{100000 \times 0.024} \approx 434\text{~}\mu F
$$

The theoretical minimum is therefore approximately:

$$
C_{\min} \approx 434\text{~}\mu F
$$

This calculation excludes ESR, ESL, manufacturing tolerance, temperature, and MLCC capacitance reduction under DC bias. It is only a lower bound on nominal capacitance. Startup, load steps, and port connection or disconnection also introduce transients, so final ripple must be assessed using component impedance curves, PCB parasitics, and measured waveforms.

Both sides use 2 × 220 µF / 50 V aluminum electrolytic capacitors, with a 10 µF / 50 V X7R capacitor, a 10 µF ceramic capacitor, and a 100 nF high-frequency ceramic capacitor in parallel. The larger capacitors support low-frequency and transient demands; the smaller capacitors reduce high-frequency impedance.

The nominal 440 µF total is only slightly above the ideal 434 µF minimum. This does not establish that the 0.1% ripple target has been met. The first revision uses ≤0.5% as its acceptance target, with 0.1% reserved for later optimization.

The capacitor bank also needs sufficient ripple current capability:

$$
I_{ripple,rated} > 3\text{~A}
$$

The ESR contribution must be included:

$$
\Delta V_{ESR} \approx \Delta I_{C} \cdot ESR
$$

### 3.3 MOSFET Selection and Losses

MOSFET losses can be estimated as:

$$
P_{MOSFET} \approx P_{cond} + P_{sw} + P_{oss} + P_{rr}
$$

Selection involves more than voltage rating and $R_{DS(on)}$. Conduction losses, switching losses, gate-drive losses, output capacitance losses, reverse recovery, thermal performance, and safe operating area (SOA) all matter.

#### 3.3.1 Conduction Losses

An approximate expression is:

$$
P_{cond} = I_{RMS}^{2}R_{DS(on)}
$$

The increase in $R_{DS(on)}$ with temperature must also be included.

#### 3.3.2 Switching Losses

A first-order estimate is:

$$
P_{sw} \approx \frac{1}{2}V_{DS}I_{D}\left( t_{r} + t_{f} \right)f_{s}
$$

At 100 kHz, switching times, gate charge, and junction capacitances have a noticeable effect on losses.

Faster switching is not always better. Faster edges shorten the switching interval and can reduce some switching losses, but they also increase the influence of PCB parasitic inductance and capacitance. Excessive voltage and current slew rates can produce spikes and ringing, making EMI harder to control.

#### 3.3.3 Gate Charge

Gate-drive power can be approximated as:

$$
P_{gate} = Q_{g}V_{GS}f_{s}
$$

This is primarily included in the gate-driver power budget.

In addition to total gate charge $Q_{g}$, I considered $Q_{gd}$ because of its relation to the Miller interval. It affects the MOSFET drain-source voltage transition and switching time, and therefore switching losses and the required gate-driver current capability.

#### 3.3.4 Output Capacitance Losses

The MOSFET output capacitance participates in each switching cycle:

$$
P_{oss} \approx \frac{1}{2}C_{oss}V_{DS}^{2}f_{s}
$$

In practice, $C_{oss}$ varies significantly with $V_{DS}$. Using the datasheet's $E_{oss}$ values and calculating $P_{oss} \approx E_{oss}f_{s}$ gives a more representative estimate.

#### 3.3.5 Reverse Recovery

During dead time, inductor current still needs a path, so a MOSFET body diode may conduct briefly. When the opposing MOSFET turns on, the stored diode charge must be removed. Reverse recovery charge $Q_{rr}$ therefore contributes to the switching process.

A larger $Q_{rr}$ can cause a more pronounced current spike during commutation. This increases losses and excites parasitics in the power loop, leading to additional ringing, transient MOSFET stress, and EMI.

I therefore compared $V_{DS}$, $R_{DS(on)}$, $Q_{g}$, $C_{iss}$, $C_{oss}$, and $Q_{rr}$, while also checking pulse current capability, SOA, thermal behavior, and package parasitics. These parameters involve trade-offs: very low on-resistance often comes with higher gate charge and capacitance. The final choice has to suit this board's voltage, power, and 100 kHz switching frequency rather than minimizing $R_{DS(on)}$ alone.

### 3.4 MOSFET Voltage Rating

I chose 60 V MOSFETs rather than selecting close to the 30 V steady-state upper limit. This provides steady-state voltage margin over the normal 18–30 V operating range. However, early PSIM mode transitions produced 40–50 V or higher transients, so the voltage rating alone does not establish adequate transient margin. Hardware measurements of drain-source voltage overshoot are needed, with layout, gate resistance, and RC snubbers used to keep peaks within a reliable range.

The four main N-channel MOSFETs are:

$$
\boxed{\text{CSD18540Q5B}}
$$

### 3.5 Switching Ringing and RC Snubbers

I have not fixed the snubber values at the schematic stage. Ringing depends on the power-loop parasitic inductance, device packages, capacitor ESL, gate resistance, and actual operating current, as well as the MOSFET itself. These effects are difficult to predict accurately before the board is built.

RC snubber footprints are provided near all four MOSFETs, using 2010 resistors and 0802 capacitors. They are initially marked DNP (do not populate).

The final $R_{snub}$ and $C_{snub}$ will be selected from measured $V_{DS}$ overshoot, ringing frequency, and decay, including their variation with load and buck or boost operation.

An RC snubber adds losses, usually dominated by charging and discharging its capacitor each cycle. Resistance mainly sets damping and peak current. The values need to be tuned together from measured ringing rather than assuming that increasing either value simply increases losses. An order-of-magnitude estimate is:

$$
P_{snub} \sim CV^{2}f_{s}
$$

## 4 Gate Drive and Current Sensing

After selecting the MOSFETs, the next task was driving four N-channel devices in two half-bridges. My initial requirements were a gate-drive voltage of approximately 10–12 V, suitable high-side operation, support for high duty cycles, dead-time control, and sufficient gate source/sink current.

### 4.1 Initial Choice of MP6528

I initially chose the MP6528 because its two-half-bridge, four-N-channel-MOSFET arrangement matched the power stage reasonably well.

Its internal VREG of approximately 11.5 V was attractive: it could supply the gate drive without a separate 12 V supply, and a $V_{GS}$ of around 10–12 V suited the CSD18540Q5B. Its high-side auxiliary charging mechanism also supported high-duty-cycle operation, and external components could adjust dead time.

### 4.2 Moving to HIP4081AIBZ

I later found that the issue was not simply whether the driver could drive four MOSFETs.

The four-switch buck-boost converter does not always operate like a conventional H-bridge. It must move between buck, boost, and the transition region while handling bidirectional power flow, CCM/DCM, and the relationship between the two bridge-leg duty cycles. In some conditions, one leg switches at high frequency while the other stays on; in the transition region, both legs may be modulated.

I wanted the controller to determine the switching states and timing of Q1–Q4 more directly, with the driver mainly providing the required gate-drive strength. The MP6528's integrated H-bridge control and protection logic was useful for conventional applications, but restricted the control flexibility I wanted here.

I therefore changed to **HIP4081AIBZ**. It still provides high-side and low-side drive for four N-channel MOSFETs in two half-bridges, but its four input controls allow more direct control of the switching states during buck, boost, transition-region, and reverse-power operation.

### 4.3 Dead Time

The upper and lower MOSFETs in each leg require dead time to prevent shoot-through if one device turns on before the other has turned off. HIP4081AIBZ allows turn-on delay to be set using external resistors on HDEL and LDEL. Hardware provides a baseline delay, while the main 200 ns dead time is generated by the STM32.

This is currently a design setting. The effective dead time must be confirmed by measuring both upper- and lower-device gate-source voltages on the actual board.

### 4.4 Current Shunt and Kelvin Sensing

Accurate inductor current measurement is important for the closed-loop controller. With a conventional two-terminal shunt, voltage drops in the pads and PCB copper can become part of the measured signal.

I selected a four-pad current shunt for Kelvin sensing. Power current passes through the Force terminals, while separate Sense terminals pick up the voltage directly across the shunt element. This reduces the contribution of high-current copper and pad contact resistance.

Ideally, the measurement follows Ohm's law: $V_{shunt} = I_{L}R_{shunt}$.

This also makes the routing clearer: the main current follows the power path, while two independent Kelvin traces run to the current-sense amplifier without sharing pickup points with the power copper.

### 4.5 Bidirectional Current Sensing

Because power flow is bidirectional, the current measurement must identify direction as well as magnitude. I selected **INA240A1DR**, using a reference voltage to place the zero-current output near midscale. The MCU determines current direction from whether the output is above or below that reference.

This signal is used for the inner current loop, DCM detection, and overcurrent protection.

## 5 Hardware Protection

Protection is not left entirely to the MCU. For faults such as overcurrent and overvoltage, I wanted hardware to shut down the power stage before software necessarily has time to respond, so the board includes an independent hardware protection path.

### 5.1 Reverse Polarity Protection

Either port may connect to an external supply or battery. Both ports therefore use **LM74502 controllers with external N-channel MOSFETs** for reverse polarity protection.

An important reason for this choice is that the LM74502 protects against reversed input polarity without inherently blocking the reverse current needed for normal bidirectional operation. Under correct wiring, the protection stage remains conductive with low losses; if the port polarity is reversed, it isolates that voltage from the power stage.

### 5.2 Bidirectional Overcurrent Protection

Overcurrent protection must cover both current directions. The INA240 output uses its reference voltage as the zero-current point, with forward and reverse currents appearing on opposite sides of that reference. Separate positive and negative hardware thresholds are therefore provided.

The current design thresholds are approximately **±8.4 A**. The MCU handles normal current regulation, but exceeding the hardware threshold directly triggers the fault logic and disables gate drive. The current loop regulates current; the hardware comparators provide protection.

### 5.3 Overvoltage Protection

Both ports have voltage monitoring and hardware overvoltage protection. The normal maximum operating voltage is around 30 V, so the protection thresholds are set outside the normal operating range. They must avoid nuisance trips from ordinary ripple or brief switching spikes without being so high that protection becomes ineffective.

Independent comparator channels supplement the MCU ADC. Once the sensed voltage meets the comparator's trip conditions, the fault signal enters the latch and gating logic to shut down the power stage. Actual immunity to spikes depends on the divider, filtering, hysteresis, and comparator propagation delay.

### 5.4 Fault Shutdown and Latching

Simply sending overcurrent and overvoltage signals to the MCU could miss a brief fault that disappears before it is sampled or processed. A hardware fault latch therefore retains the event.

Forward and reverse overcurrent signals are combined into `FAULT_OC`, and port A and port B overvoltage signals into `FAULT_OV`. These are then combined into `FAULT_SET`. A **74LVC2G02 dual NOR gate** implements the SR latch. When `FAULT_SET` is asserted, `NOT_FAULT` goes low and remains latched even after the original overcurrent or overvoltage condition disappears.

`NOT_FAULT` participates in driver enable and hardware PWM gating, allowing the logic to block PWM to the gate driver without waiting for an MCU shutdown routine.

The power stage does not restart automatically when the fault disappears. A new `FAULT_RESET` is required. A manual reset button is also provided for board testing, helping avoid repeated startup and shutdown while a short circuit or overcurrent condition remains unresolved.

The protection chain consists of comparator detection, fault-state latching, and PWM gating. The MCU reads the status and decides when reset is permitted, reducing dependence on software response time and continued software operation.

### 5.5 ESD and Interface Transient Protection

Ports A and B connect directly to external supplies, batteries, or laboratory equipment, so ESD and interface transients must be considered alongside overvoltage and reverse polarity protection.

These events can be too brief for MCU-based or ordinary voltage monitoring to handle. Protection devices should therefore sit close to the external interfaces, with short connections to the interface and ground so transient current can be diverted before reaching sensitive circuitry.

The controller-to-power-board interface carries four PWM signals, fault and reset signals, current and voltage measurements, and temperature signals. Because these ultimately reach MCU or logic inputs, ESD protection is provided at the interface to protect the 3.3 V logic during connection, debugging, and external electrostatic discharge.

**PESD3V3L4UG** devices clamp multiple signals, including PWM, FAULT, FAULT_RESET, current and voltage sensing, and temperature sensing. Most interface paths also include approximately **100 Ω** series resistors to limit transient current and soften fast edges and interface noise. The external 3.3 V supply has separate ESD protection.

This stage handles fast electrostatic and transient disturbances at the control interface. Its role is to protect the MCU and logic signals, while overvoltage, overcurrent, and reverse polarity protection address abnormal power-stage conditions.

### 5.6 Inductor Temperature Monitoring

In addition to checking rated and saturation current, I included inductor temperature sensing. Inductor losses include both DCR-related copper losses and core losses at 100 kHz, and vary with current, ripple, and operating mode.

Datasheet-based temperature estimates alone are therefore insufficient. The temperature signal is sent to the MCU so the actual operating temperature can be monitored, particularly during sustained high-power operation or limited cooling.

### 5.7 Protection of the 12 V Gate-Drive Supply

The gate drivers use a separate 12 V supply. Although its power is modest, an abnormal supply voltage affects the switching of all four main MOSFETs, so the external supply is not connected directly to the driver.

The input first passes through a fuse, a Schottky diode, and a TVS. The fuse provides isolation after a short circuit or sustained overcurrent, the Schottky diode provides reverse polarity protection, and the TVS clamps connection transients. A downstream TPS26620 eFuse provides undervoltage and overvoltage protection and current limiting before supplying the gate-drive circuitry.

## 6 Measurement Interfaces and Grounding

### 6.1 Voltage Measurement

The MCU measures both port voltages as well as inductor current. These measurements support the outer voltage loop, buck/boost region selection, and operating-state monitoring.

Both ports use resistor dividers feeding ADC inputs. The current **82 kΩ / 6.8 kΩ** divider gives a ratio of approximately:

$$
\frac{6.8}{82 + 6.8} \approx 0.0766
$$

A port voltage of around 30 V therefore produces approximately 2.3 V at the MCU ADC, leaving some margin. Simple RC filtering at the sensing inputs attenuates part of the switching noise.

### 6.2 External Control Interface

A dedicated interface connects the power board to the MCU board. It carries four PWM signals, fault status, reset, current and voltage measurements, and MOSFET and inductor temperature signals.

The 3.3 V logic supply passes through a ferrite bead at the interface and has local decoupling. Control-signal ESD protection is also grouped near the interface. This makes the supply and signal boundaries between the two boards easier to inspect independently.

### 6.3 Ground Partitioning and Return Paths

Return paths are divided by function into PGND, DRV_GND, and GND. PGND carries the main high-current power return, DRV_GND serves the 12 V gate-drive circuitry, and GND is the reference for sensing, logic, and control signals.

Most power components are on the top layer, with PGND directly below the power section. Keeping outgoing and return switching-current paths close reduces high $di/dt$ loop area and parasitic inductance. PGND covers roughly the upper half of the board, while avoiding the control and sensing areas, especially PWM enable logic and Kelvin sensing, to keep power return current away from sensitive signals.

The fourth-layer GND plane covers most of the board and provides a reference for control, sensing, and other small signals. DRV_GND is mainly on the top layer and parts of the bottom layer, with wide copper connections to GND. Because the fourth-layer plane still extends beneath part of the power section, capacitive coupling from switching nodes and common-mode noise need particular attention during first-board testing.

The three ground regions are not electrically isolated. They meet at a designated point near the lower-left corner of the PCB. The aim is to keep power, gate-drive, and small-signal return currents primarily within their own regions, reducing power current through the sensing and control areas. This connection point is a first-revision layout choice and will be reassessed using measured ground bounce, gate-drive return behavior, and current-sense noise.

### 6.4 Status Indicators

The board includes indicators for both port bus voltages, the 3.3 V logic supply, the 12 V gate-drive supply, driver enable, and the fault latch.

The port A and B indicators connect to their respective buses through 22 kΩ resistors to show whether each side is energized. The 3.3 V and 12 V supplies have separate indicators. Driver-enable and fault indications make basic states visible without repeatedly checking them with a meter or oscilloscope.

These LEDs use modest current to reduce unnecessary power consumption. They are intended to make startup, protection, and fault-reset debugging easier by showing where operation has stopped.

## 7 PCB Layout

The main power board uses six layers. Most power components are on the top layer; internal layers provide power return paths, signal routing, and logic supply distribution. Sensing and protection circuits are placed on the bottom layer.

| Layer | Main function |
|---|---|
| Top | Main inductor, MOSFETs, power paths, reverse polarity protection, gate drive, control interface, current-sense core, and status indicators |
| Layer 2 | PGND, mainly beneath the power section, avoiding PWM enable and Kelvin sensing areas |
| Layer 3 | Kelvin, PWM, ADC, and other signal traces |
| Layer 4 | GND reference plane covering most of the board |
| Layer 5 | 3.3 V logic supply distribution |
| Bottom | ADC, fault latch, temperature sensing, and overcurrent/overvoltage shutdown circuits |

### 7.1 Power and Control Regions

The upper part of the PCB contains the power stage, MOSFETs, inductor, power capacitors, reverse polarity protection, and gate-drive circuitry. The lower part contains the ADC, protection logic, interfaces, and other small-signal circuits.

This keeps regions with high $di/dt$ and high $dv/dt$ away from low-level sensing signals. Top-layer power nets use copper pours with short high-frequency current loops. For SW_A and SW_B, copper area is chosen to meet current and temperature requirements while limiting parasitic capacitance to reference planes and common-mode EMI.

### 7.2 Gate-Drive Layout

HIP4081AIBZ and its gate traces are placed close to the MOSFETs. Although gates do not carry the main power current, they carry substantial current pulses during switching, so the gate and source return loops must be small.

Gate resistors sit close to the MOSFETs. If hardware testing reveals excessively fast edges or ringing, these resistors can be adjusted without redesigning the whole PCB.

The RC snubber footprints are also close to the MOSFETs. They remain DNP until measured $V_{DS}$ ringing provides a basis for selecting values.

### 7.3 Kelvin and ADC Routing

The shunt's power pads sit directly in the main current path. Its Sense pads have separate traces, routed as a pair rather than sharing power copper.

Each gate-drive connection and its corresponding source return form a local loop. The outgoing path runs through the gate resistor to the MOSFET; the return connects to that device's source reference. Adjacent vias are used where layer changes are needed, avoiding a detour through the control ground.

The two Kelvin traces leave independent shunt sensing pads and run as a pair on layer 3 toward the current-sense amplifier, away from switching nodes and gate traces. Their pickup points are not shared with the main power copper.

The top-layer bidirectional current-sense circuitry and bottom-layer ADC and protection shutdown circuits are arranged close together, with nearby vias to shorten analog signal paths. Voltage and temperature sensing are also concentrated in the control area, with only the necessary measurement connections entering the power section.

### 7.4 Main Power Stage Layout

The main power path follows port A → bridge A → inductor → bridge B → port B. The four main MOSFETs are grouped around their respective bridge legs, keeping connections between switching nodes, the inductor, and bus capacitors short to reduce high $di/dt$ loop area and parasitic inductance.

Reverse polarity MOSFETs and input/output capacitors sit near each port. Incoming current passes through protection before entering the main power circuit. This limits long high-current paths, while keeping bus capacitors near the bridge legs to supply switching-current pulses locally.

Main power connections use copper pours rather than narrow traces. A_BUS, B_BUS, and PGND prioritize current capability and temperature rise, with layer 2 PGND providing a nearby return path. SW_A and SW_B use only the area needed for current and thermal requirements to avoid unnecessary capacitance and EMI.

The 33 µH inductor is placed between the bridge legs along a nearly straight power path. The current shunt is close to the inductor and the main current path, with independent Kelvin connections leading to the sensing circuitry.

The first revision includes layer 2 PGND keepouts beneath SW_A, SW_B, and adjacent high-dv/dt regions to reduce capacitance between switching nodes and PGND. Layer 4 GND remains largely continuous. This is a first-revision compromise; common-mode noise and switching measurements will determine whether keepouts should extend across more layers.

The board includes 1 mm diameter test pads. After receiving the bare boards, I found these too small for stable oscilloscope probing. A later revision will enlarge the test points and add suitable ground-spring connection points.

## 8 Control Strategy

### 8.1 Outer Voltage Loop and Inner Current Loop

The controller uses cascaded voltage and current loops. In the current simulation, both update every 10 µs, corresponding to a 100 kHz control update rate. The target current-loop bandwidth is approximately 5 kHz, and the voltage-loop target is approximately 500 Hz, about one decade apart. A soft-start ramp on the voltage reference reduces transient current caused by a large reference-to-output mismatch during startup or mode changes.

### 8.2 Buck and Boost Operating Regions

For power flow from A to B with $V_{A} > V_{B}$, the converter operates in the **buck region**. Bridge A is PWM-modulated, while bridge B mainly stays on to conduct inductor current to port B. The ideal duty-cycle relationship is approximately:

$$
D_{\text{Buck}} \approx \frac{V_{B}}{V_{A}}
$$

When $V_{A} < V_{B}$, the converter operates in the **boost region**. Bridge A mainly stays on, while bridge B is PWM-modulated to store and release energy through the inductor. The ideal relationship is:

$$
D_{\text{Boost}} \approx 1 - \frac{V_{A}}{V_{B}}
$$

When power reverses, the input and output roles swap, but the physical leg selected for high-frequency modulation still depends on the port voltage relationship. In this design's single-leg modulation regions, when port A is higher than port B, bridge A switches for both forward buck and reverse boost operation, with the B-side high-side device held on. When port A is lower than port B, bridge B switches for both forward boost and reverse buck operation, with the A-side high-side device held on. The controller also adjusts the current-reference direction and the PWM and synchronous-rectification timing.

A fixed buck-only or boost-only mode cannot cover the full input/output voltage range. Modulating all four MOSFETs at high frequency in every condition is possible in principle, but introduces additional switching losses, gate-drive losses, and EMI.

I therefore use single-leg modulation when the port voltages differ sufficiently, and coordinated modulation of both legs in the four-switch transition region when $V_{A}$ and $V_{B}$ are close. This covers step-up and step-down operation while reducing unnecessary switching.

### 8.3 Transition Region and Dual-Duty-Cycle Modulation

Away from the transition region, one leg switches and the other provides a continuous conduction path. As $V_{A}$ and $V_{B}$ approach one another, directly switching between buck and boost would abruptly change device states and duty cycles. A dedicated buck-boost transition region is therefore used near $V_{A} = V_{B}$.

The transition region uses dual-duty-cycle control with a fixed duty-cycle difference, as referenced in US Patent 7,804,283 B2. Defining $D_{\text{buck}}$ and $D_{\text{boost}}$, the A → B switching cycle has three main states:

| State | Conducting devices | Inductor voltage | Duration |
|---|---|---|---|
| Energy storage | Q1 + Q4 | $V_{A}$ | $D_{\text{boost}}T_{s}$ |
| Energy transfer | Q1 + Q3 | $V_{A} - V_{B}$ | $\left( D_{\text{buck}} - D_{\text{boost}} \right)T_{s}$ |
| Freewheeling | Q2 + Q3 | $- V_{B}$ | $\left( 1 - D_{\text{buck}} \right)T_{s}$ |

The three durations add up to one switching period. Inductor volt-second balance gives:

$$
\frac{V_{B}}{V_{A}} = \frac{D_{\text{buck}}}{1 - D_{\text{boost}}}
$$

The two duty cycles maintain a fixed difference in the transition region:

$$
D_{\text{buck}} - D_{\text{boost}} = K
$$

The selected value is:

$$
K = 0.70
$$

With duty-cycle limits:

$$
D_{\min} = 0.10,\ D_{\max} = 0.90
$$

At 100 kHz, the period is 10 µs, so a 10% duty cycle corresponds to 1 µs, leaving useful pulse width with the 200 ns dead-time setting.

When the port voltages are nearly equal:

$$
V_{A} \approx V_{B}
$$

The central operating point is:

$$
D_{\text{buck}} = 0.85,\ D_{\text{boost}} = 0.15
$$

Which satisfies:

$$
0.85 - 0.15 = 0.70
$$

This relationship coordinates the two duty cycles within the transition region. Entering or leaving the region still requires appropriate duty-cycle mapping, controller-state handling, and PWM timing. The fixed difference and mode hysteresis alone do not guarantee transitions without duty-cycle jumps or current transients. Current transition behavior is shown in Section 9.

### 8.4 Switching Between Buck, Transition, and Boost Regions

Region selection uses the filtered port voltages:

$$
\varepsilon = \frac{V_{A} - V_{B}}{\max\left( V_{A},V_{B} \right)}
$$

When $V_{A}$ is sufficiently higher than $V_{B}$, bridge A switches at high frequency and bridge B provides the continuous conduction path. When the voltages are close, both legs participate in the transition region. When $V_{A}$ is sufficiently lower than $V_{B}$, bridge B switches and bridge A provides the continuous conduction path.

Entry and exit thresholds differ by one percentage point:

| Current state | Transition condition | Next state |
|---|---|---|
| A switching; B held on | $\varepsilon \leq + 10\%$ | Transition region |
| Transition region | $\varepsilon \geq + 11\%$ | A switching; B held on |
| B switching; A held on | $\varepsilon \geq - 10\%$ | Transition region |
| Transition region | $\varepsilon \leq - 11\%$ | B switching; A held on |

This hysteresis helps prevent repeated state changes when ripple or sensing noise affects $V_{A}$ and $V_{B}$ near a boundary.

### 8.5 PWM and Dead Time

The STM32 generates four PWM signals according to the operating region and power direction. Upper- and lower-device commands in each leg are complementary, with a 200 ns dead-time setting. `NOT_FAULT` and driver enable also participate in hardware gating so a fault can force the gate-driver inputs into the off state.

The 200 ns value is a controller setting. Effective dead time must be verified at the MOSFET pins by measuring the interval between one device turning off and the other turning on.

The table below describes basic A → B modulation. For B → A, step-up or step-down operation must be determined from the actual input and output voltages. The physical switching leg still follows the voltage-region rules in Section 8.4; A and B must not simply be swapped. Held-on states apply during normal energy transfer; zero-current intervals at light load are handled by the DCM strategy.

| Region | High-frequency switching leg | Device held on | Condition |
|---|---|---|---|
| Buck | Bridge A: Q1, Q2 | B-side high-side Q3 | Port A voltage above port B |
| Transition | Bridges A and B | Neither side permanently held on | Coordinated dual-duty-cycle modulation |
| Boost | Bridge B: Q3, Q4 | A-side high-side Q1 | Port A voltage below port B |

### 8.6 CCM and DCM

Rated-load operation is mainly CCM, with DCM permitted at light load. The controller uses filtered inductor current and zero-current detection to determine the conduction state, adjusting synchronous rectification near zero current to avoid unintended reverse current.

The current DCM entry and exit thresholds are approximately 1.5 A and 2.0 A, with the condition required to persist for three switching cycles. The provisional zero-current detection threshold is 0.08 A.

Light-load simulations show discontinuous current intervals, but small reverse oscillations after zero crossing still need refinement. DCM is therefore reported as a preliminary simulation result, not completed hardware validation.

## 9 PSIM Simulation Validation

### 9.1 Simulation Model

![PSIM model of the bidirectional four-switch buck-boost converter](assets/design-report/psim-model.png)

Figure 1. PSIM model of the bidirectional four-switch buck-boost converter.

The first stage used ideal MOSFET and inductor models to check bidirectional power flow, steady-state parameters, and the power-stage topology.

The second stage introduced Level 2 MOSFET models, a Level 1 inductor model, 12 V gate drive, complete PWM logic, transition-region control, and the main capacitor values from the schematic. This model helps identify control and device-stress issues, but does not replace PCB parasitic extraction or hardware testing.

### 9.2 Forward Power Flow Across Operating Regions

![Bus voltage response as port A changes from 30 V through 24 V to 18 V](assets/design-report/forward-voltage-transition.png)

Figure 2. Port A and port B bus voltage responses during the 30 V → 24 V → 18 V sequence at port A.

This waveform includes startup and operating-point transitions. Port B falls to approximately 11 V early in the sequence. It demonstrates operation across buck, transition, and boost regions, but is not evidence that final voltage-regulation requirements have been met. Separate startup, input-step, and steady-state windows will be provided later.

### 9.3 Inductor Current Transient Response

![Inductor current transient response during the combined forward-power test](assets/design-report/inductor-current-transient.png)

Figure 3. Inductor current during the combined forward-power operating sequence.

Inductor current changes with input voltage, load, and operating region. There are clear transient peaks during transitions. These results mainly demonstrate continued state-machine operation and the intended power-flow direction; peak current and recovery time still need to be quantified separately for each condition.

### 9.4 Heavy-Load CCM Inductor Current

![Heavy-load CCM inductor current](assets/design-report/ccm-inductor-current.png)

Figure 4. Steady-state inductor current in heavy-load CCM operation.

Current remains continuous in the selected steady-state window. The approximately 1.4 A peak-to-peak ripple is broadly consistent with the theoretical calculation for the 33 µH inductor.

### 9.5 Light-Load DCM Inductor Current

![Light-load DCM inductor current](assets/design-report/dcm-inductor-current.png)

Figure 5. Inductor current in light-load DCM operation.

Zero-current intervals are present, but small negative oscillations remain after zero crossing. This is a preliminary DCM result. The zero-current detection threshold, mode exit conditions, and synchronous MOSFET turn-off timing still need refinement.

### 9.6 Reverse 24 V to 30 V Boost Operation

![Reverse 24 V to 30 V boost response](assets/design-report/reverse-boost-24-to-30.png)

Figure 6. Boost response with power flowing from port B to port A, nominally 24 V → 30 V.

Port B supplies port A, with a nominal B-side voltage of 24 V and an A-side target of 30 V. VBUS_A (green) and VBUS_B (red) are bus voltages in volts. iL (blue) is the inductor current, while I_CONNECTOR_A (light orange) and I_CONNECTOR_B (orange) are port currents, all in amperes. The horizontal axis is time.

Positive inductor current is defined from A to B, so iL is negative during reverse power transfer. The waveform shows the A-side voltage recovering to approximately 30 V after a transient. The actual B-side voltage is given by the red trace; a nominal 24 V does not mean it remains exactly 24 V throughout the run.

### 9.7 Reverse 24 V to 18 V Buck Operation

![Reverse 24 V to 18 V buck response](assets/design-report/reverse-buck-24-to-18.png)

Figure 7. Buck response with power flowing from port B to port A, nominally 24 V → 18 V.

Port B supplies port A, with a nominal B-side voltage of 24 V and an A-side target of 18 V. Trace names, units, and the inductor-current sign convention are the same as in Figure 6. The A-side voltage initially falls to approximately 10 V and then recovers to approximately 18 V. Negative inductor current indicates B → A energy transfer.

This plot illustrates reverse-buck transient behavior. It does not establish that voltage-regulation accuracy requirements are met throughout the sequence.

### 9.8 Gate-Source Voltages of the Four MOSFETs

![Gate-source voltage waveforms for Q1, Q2, Q3, and Q4](assets/design-report/mosfet-gate-voltages.png)

Figure 8. Gate-source voltage waveforms for Q1–Q4.

At this time scale, the waveform confirms operation of the four PWM channels and gate-drive logic, but cannot verify a 200 ns dead time. The switching edges must be examined on an expanded time axis, followed by measurements at the actual MOSFET pins.

## 10 First-Revision Hardware and Test Plan

### 10.1 Hardware Status

![Main power PCB front](assets/design-report/power-board-front.jpeg)

![Main power PCB back](assets/design-report/power-board-back.jpeg)

Figure 9. Front and back of the bare main power PCB.

A two-layer MCU adapter board was also designed to connect the controller to the main power board. It includes expansion control connections and an OLED interface. The display can later show power-flow direction, efficiency, fault status, and key measurements. The main power PCB and adapter PCB are currently unpopulated; component assembly and power-up are still pending.

![MCU adapter and controller boards](assets/design-report/controller-boards.jpeg)

![MCU adapter and OLED display](assets/design-report/controller-oled.jpeg)

![MCU adapter assembly arrangement](assets/design-report/controller-assembly.jpeg)

Figure 10. MCU adapter and OLED mounting arrangement.

### 10.2 Initial Power-Up Sequence

1. With power disconnected, check component orientation, soldering, and port polarity, and measure resistance from the main supply nets to ground.
2. Apply current-limited power to the 3.3 V logic supply only. Check the supply, fault latch, and reset state while keeping PWM disabled.
3. Apply current-limited power to the 12 V gate-drive supply. Check the fuse, TVS, TPS26620, and static driver outputs.
4. Keep the main power bus disconnected or at a safe low voltage. Apply PWM and measure all four MOSFET gate-source voltages, complementary timing, and effective dead time.
5. Use a low-voltage, current-limited supply and resistive load to verify buck and boost operation in both A → B and B → A directions, sensing polarity, and protection gating.
6. After all low-voltage checks pass, increase voltage and power gradually, recording drain-source voltage overshoot, inductor current, ripple, efficiency, and temperature rise.
7. Use differential probes or an equivalent safe measurement arrangement for SW_A, SW_B, and high-side gate-source voltages. Do not attach an earth-referenced oscilloscope ground clip directly to a switching node.

### 10.3 Initial Measurements

- $V_{GS}$
- $V_{DS}$
- Effective dead time
- SW_A / SW_B
- Inductor current
- Output ripple
- Current-sense signal
- Port A and port B bus voltages
- NOT_FAULT and driver enable
- Overcurrent and overvoltage protection response
- Inductor and MOSFET temperatures

### 10.4 Snubber Tuning

Leave the RC snubbers unpopulated initially. Use low-inductance probing to record drain-source voltage overshoot, ringing frequency, and decay without a snubber. Start with a small capacitance and adjust the resistance, comparing overshoot, ringing, device temperature rise, and efficiency across buck, boost, and different load conditions before choosing final values.

### 10.5 Further Validation

Pending measurements include efficiency, output ripple, voltage-regulation accuracy, load-step response, power-flow reversal, CCM and DCM behavior, temperature rise, and protection response.

These performance metrics have not yet been validated on hardware. Once the test setup is available, I will record original waveforms, test conditions, and calculation methods under consistent operating conditions and continue updating the project results.
