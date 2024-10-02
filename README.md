# Spice-it (Smart Spice Dispensor)

Let's face it, countless young and novice chefs (i.e. University/College students) struggle with achieving the perfect balance of spices in their dishes, leading to inconsistent flavours and wasted ingredients. Spice-it is a smart and automated kitchen appliance with the objective to streamline the cooking process by accurately dispensing seasonings to the user’s preferences and preset recipes. This eliminates the need for manually pouring multiple spices while keeping your hands clean.

The grand undertaking of designing, building, and testing of the first prototype of Spice-it is captured through the hard work of four Electrical Engineering students at the University of Waterloo through their Final Year Design Project or Capstone.

At its core, Spice-it can be viewed through the lens of 3 major subsytems :
- Power Electronics
- User Interface/Software Logic
- Spice Dispense/Mechanical Design

  The block diagram below helps to give more perspective into how these subsystems are interconnected
  
![image](https://github.com/user-attachments/assets/539fb2f7-06ef-4992-ae30-3b2013ad00fd)

_Figure 1. Spice-it System Block Diagram_

# Power/Circuit Electronics

My contributions towards Spice-it have been focused in the design and development of the Power/Circuit Electronics. It is the first major subsystem found in Spice-it whose ultimate objective is to provide the correct amount of power to satisfy the demands of the electrical components in our system through safe and controlled means. Each major block of the subsystem plays a crucial and essential role in the overall function and are described in further detail below.

## Transformer

Starting with a 120 VAC to 12 VAC transformer, Spice-it taps into the electrical grid to receive a reliable and uninterrupted supply of power. This transformer is selected based on the safety requirements (CSA, UL, ULC, ESA certified), availability to order, and ease of use in operation.

## Power Switch

In order to allow for no current draw in the off-state of the Spice-it device, a power button is included. Given the 12 VAC input waveform, a stainless steel latching push button is used which can support 12-24V AC or DC. This is clearly suitable for our application of 12 VAC. Also of note is that the button supports a current of up to 3 A which is enough to support the calculated current draw of the device. Information on the current draw specifics is found later in the report. The output of the button is fed through 18 American Wire Gauge (AWG) wire, as per expected current draw, which then is terminated and fed into a printed circuit board (PCB) header on the power electronics PCB. 

## Device Current Draw
The following table is obtained from controlled tests of each electrical component in our system. Each component exists in a 5 V or 3.3 V circuit and contributes to that circuit’s overall current draw. Considering the current draw ratings for the components used for the device allows for the proper protection planning and resource adequacy considerations.

_Table 1. Measured and rated values for device current draw_

| Component  | Individual Motor | Microcontroller  | Digital Display |  Max Total Current | 
| ------------- | ------------- | ------------- | ------------- | ------------- |
| 5 V Output  | Operating measured: 110 mA  Max measured: 222 mA  | N/A  | N/A  |  Max: 222 mA  Operating: 110 mA  |
| 3.3 V Output  | N/A  | Max: 500 mA  Operating: 80 mA  Measured: 34 mA  | Max: 36.5 mA  Operating: 30.5 mA  | Max: 536.5 mA  Operating:110.5 mA  |

  ## AC-DC Converter
Following the power switch, the process of rectifying the AC into DC and converting the DC into the target bus voltages begins. The first step involves a Single-Phase Diode Bridge Rectifier that is designed to have a maximum output voltage ripple of 5%, illustrated in Figure 2. This value is obtained through a capacitor with the rating of 5.56mF by using the design equation C_d=  T/(2R (ΔV_o)/V_o ) with the values T = 60^(-1)  s,R=30 Ω,and  (ΔV_o)/V_o =0.05. It is important to maintain an output voltage ripple of less than or equal to 5% to ensure that the 555 timer powering the buck converter later in the circuit creates a stable clock [3], [4]. This setup is simulated (Figure 3) using PSIM to ensure that theoretical calculations are within an acceptable tolerance in a simulated environment.

![image](https://github.com/user-attachments/assets/fdcba489-a18f-41d7-a620-42f102df79d0)

_Figure 2. Diode Bridge Rectifier Circuit Schematic_

![image](https://github.com/user-attachments/assets/ded51557-b11f-4244-8d79-031c547f1e9f)

_Figure 3. Diode Bridge Rectifier Simulation_

## Voltage Regulator

After being rectified, the next step in the AC-DC converter process is the use of a voltage regulator integrated circuit (IC) to allow for the reduction of the rectified 12 V input to a 5 VDC output (Figure 4). The regulator must also output enough current to run the device within the expected range. With these considerations in mind, the LM7805CT/NOPB voltage regulator is chosen [5]. This regulator has an input voltage range of 7.5V-35V and an output current of 1.5 A. These values overcompensate for the 12 V input voltage and the max expected current of approximately 758.5 mA required for proper operation. The datasheet also recommends the addition of a 0.22 uF capacitor on the input if it is far from the power supply filter and a 0.1 uF capacitor on the output for a better transient response. The input capacitor is not used here as the regulator is located right beside the power supply filter. The output capacitor, however, is used as suggested to obtain a better transient response.

![image](https://github.com/user-attachments/assets/7d8ceb21-ae1b-4e56-ba3d-ca23afe5e44b)

_Figure 4. Schematic of the voltage regulator circuit_

## Design of Buck Converter
Following the regulator stage, a buck converter (Figure 5) drops the 5 V signal to 3.3 V to power the circuits of the microcontroller unit and associated peripherals. To ensure a smooth DC output that exceeds specifications, the buck converter’s design allows for a maximum current ripple of 5% and a maximum current ripple of 10% (Figure 6). The designed elements of the buck converter, namely the inductor and capacitor, are 141 uH and 547 nF respectively. These components values are derived from the following design equations: L =  (V_o (1-d))/(f_s I_L  (ΔI_(L:))/I_L ),C=(1-d)/(8L〖f_s〗^2  (ΔV_o)/V_o ) V_o=3.3V,d=3.3/5=0.66,f_(s )=105kHz,I_L=758.5 mA,(ΔI_(L:))/I_L =0.10,L=141μH,and  (ΔV_o)/V_o =0.05
To validate the theoretical calculations, PSIM is used to simulate ideal circuit behavior (Figure 6).

![image](https://github.com/user-attachments/assets/0904e4ed-6abf-4dd6-bba1-9f7dff29c6af)

_Figure 5. Schematic of the DC-DC Buck Converter_

![image](https://github.com/user-attachments/assets/c30aebf7-0025-449b-8603-d617baf82029)

_Figure 6. DC-DC Buck Converter Circuit Simulation_

## Buck Converter Control Signal
The buck converter control signal is required to be a pulse width modulation (PWM) signal with a duty cycle which is designed based on the desired output voltage. The equation for the duty cycle of a buck converter is Vout / Vin = d. For an input of 5 V and output of 3.3 V, the duty cycle becomes 0.66. Therefore, generation of a 5V PWM signal with a duty cycle of 0.66 is required. To supply this, a 555 timer (COM-16473) (Figure 7) is used with a corresponding circuit.This chip was specifically selected due to it supporting the required voltage input of 5 V. The accompanying circuit design is created to match the manufacturer provided circuit for astable operation which is equivalent to the PWM signal required. Using the provided duty cycle equation for this setup, (R1+R2)/(R1+2R2) = d, the values of 910 Ω and 1 KΩ are selected for resistor R1 and R2 respectively. To start, a capacitor value of 4.7nF is selected which gives a frequency value of 105KHz according to the frequency equation of the 555 timer which is1.44/((R1+2R2)(C)). Running a PSIM simulation of the circuit gives the following values. 5Vout = 5V, 3.3Vout = 3.73V, f = 133KHz, and d = 0.595. Here, the 3.3V output is too high so the duty cycle is decreased by decreasing R1 to 800Ω. Running the simulation once again, the following values are observed. 5Vout = 5V, 3.3Vout = 3.26, f = 133KHz, d = 0.465 (Figure 8). These values match with the desired output so R1, R2 and C are chosen accordingly for the design.

![image](https://github.com/user-attachments/assets/335a7062-fb63-490f-a3cf-72bffa7212b7)

_Figure 7. Schematic of 555 timer circuit_

![image](https://github.com/user-attachments/assets/e1007f69-3e25-4b6b-ada3-c515d0e68365)

_Figure 8. Simulation 5 V and 3.3 V outputs_

## Circuit Protection

The PWRDST block handles circuit protection under undesirable circumstances. The system is simply composed of fuses located before each power drawing device. These include all servo motors, the microcontroller, and 3.3 V peripherals (i.e., user inputs). The addition of 3 spare fuses is made to support any future add-ons to the circuit. For the current design, these are left unpopulated as they are not used. For the motor outputs, slow blow fuses are chosen to support the inrush current associated with motors. Going off of the measured operating current of 110 mA at 5 V and the maximum measured current of 222 mA at 5 V, the fuse ratings are sized up to 500 mA. For the microcontroller, the datasheet lists up to 500 mA of current draw. However, a smaller current draw range of 30-80 mA was measured under several tests. Given the availability of excess current before reaching the bottleneck of the 1.5 A output from the voltage regulator (taken from the datasheet), a fuse value of 600 mA is used to support significant additional functionality which might increase the current draw under normal use. Additionally, this fuse is chosen to be slow blow because the device also has non-insignificant inrush current associated with it due to it needing to charge capacitors as well as having voltage regulators on its outputs. In the case of the 3.3 V peripherals, the expected current draw is very low at around 40 mA at max drive strength based on the datasheet. The fuse rating is sized at 50 mA to support both the max drive strength as well as lower values which are more realistic. For this case, low inrush current is expected due to the nature of the microcontroller input pin (meant as an analog or digital read rather than a load). As such, a fast blow fuse is appropriately selected.

![image](https://github.com/user-attachments/assets/d681feaf-18e6-4563-9fca-670f6754a1d3)

_Figure 9. Schematic of the power electronics circuit protection block_

## Additional PCB Design Considerations
It is imperative that the PCB is able to support the brunt of the load current in most sections except for the 555 timer circuit and anything after the fuses which have less current. Specifically, the bottleneck of this path of concern is the voltage regulator which supports at max 1.5 A. Thus, the PCB trace widths on this path are designed to be able to support this. Selecting 1 oz copper for the board due to its manufacturability, a mathematical model is used to find appropriate trace widths. See references [6] and [7] for more information on the models used. Finding the width for an external layer carrying 2 A with a 10°C increase above ambient temperature yields 0.92 mm on the upper end (worst case from both models). As such, a 1 mm trace width is selected for the main current path. For the fused traces, it is acceptable to simply size the widths to handle the rated current of the fuse. Following the same method, trace widths are chosen as 0.3 mm for each motor, 0.4 mm for the microcontroller, and 0.3 mm for the peripherals (to ensure adherence to the minimum manufacturing trace width requirement). Spares are chosen to have a trace width of 0.3mm to allow for additional motors in the future if needed. According to the 555 timer datasheet, the maximum supply current is 6mA, so each trace in this system can be designed around this value. Since the current is very small, the trace width is chosen at 0.3 mm for the same reason as for the peripherals.

The power input connector, which is connected after the power button, is rated to overcompensate the 2 A of current resulting from the regulator and additional losses in the conversion process. All other connectors and their associated headers are chosen to just overcompensate the fused ratings of the traces they connect to. Additionally, wires connecting to output connectors are chosen at 24AWG based on these ratings. 

The AC-DC converter and circuit protection blocks are designed into a PCB to reduce the amount of wires used in the system. Instead, these connections will be handled by traces on the PCB. Additionally, the microcontroller is embedded into header pins on the PCB for the same purpose.

