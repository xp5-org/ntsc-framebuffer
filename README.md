NTSC framebuffer built around 74x cmos logic gates and an 128k x 8bit sram module 

** this is incomplete and not yet built, in-progress **

- 8 bits for color + brightness
- 17 bits address
- transfer data in during vblank period


Tasks remaining:
  - complete memory addressing circuitry
  - check gate timings for brightness & color control paths
  - place adjustment potentiometers for setting PNP transistor bias
  - distribute ceramic capacitors
  - add debug testpads onto pcb


# Table of Contents
- [Current PCB Layout](#current-pcb-layout)
- [H&V Counter](#hvcounter)
- [HRESET](#hreset)
- [VRESET](#vreset)
- [HBLANK](#hblank-hsync)
- [VBLANK](#vblank-vsync)
- [COLORBURST](#colorburst)
- [COLORSELECT](#colorselect)
- [BRIGHTNESS](#brightness)
- [MEMORYANDBUS](#memoryandbus)
- [Output Amplifier](#outputamp)
- [Components selection](#components)
- [Simulating](#simtools)




# Current PCB Layout
![pcb_infographic](https://github.com/user-attachments/assets/7c2ed00b-3327-469c-a76e-db1d633f8326)
<br>
![pcb_infographic_3d](https://github.com/user-attachments/assets/8ca9f093-52e4-4b77-a9d0-3694e24b43e5)
<br><br><br><br><br><br><br><br> 

# hvcounter
This is the core of the circuit, its two 4040 counters, one driving the other. the first counts out 454 which is the number of inactive + active horizontal pixels per row. the output is binary, and with enough AND gates alongside numerous D type flipflops, specific events can be timed out and set to form the sync timings needed

<img width="1116" alt="hv_counter_schematicblock" src="https://github.com/user-attachments/assets/ac2bd6a3-3dfa-4c98-8c49-ba28c4ea0735" />


<br><br><br><br><br><br><br><br> 

# HRESET
<img width="1052" alt="Screenshot 2025-03-30 at 7 25 06 PM" src="https://github.com/user-attachments/assets/34855e6e-d72b-4c5e-9eff-090c872445bc" />
256 + 128 + 64 + 4 + 2
counts out 454 horizontal & sets HRESET flag to high 5V
and on the next clock pulse from not the timer we are resetting but the actual clock source


<br><br><br><br><br><br><br><br> 

# VRESET
<img width="1226" alt="Screenshot 2025-03-30 at 7 37 49 PM" src="https://github.com/user-attachments/assets/7eef5ace-3dd1-475a-9789-0cfee36c96b9" />
vreset is 1V + 4V + 256V for 261 vertical lines before resetting the counter

vertical reset is checked at the end of the horizontal scanline, so we use _HRESET as the clock for vreset - the condition of 261V lines is set only when _HRESET is ready

# HBLANK-HSYNC
<img width="1029" alt="Screenshot 2025-03-30 at 7 38 18 PM" src="https://github.com/user-attachments/assets/17e7817a-200d-4ac1-a96d-949750503598" />
<img width="1030" alt="Screenshot 2025-03-30 at 7 38 24 PM" src="https://github.com/user-attachments/assets/3694a95f-42bc-481d-b5ba-69d04ee7cfa1" />

the circuit starts off with some DC bias to give a sync tip from the "previous" scanline, and then at H2 + H8 HBLANK-ENABLE flop is cycled on. When this is on, elsewhere in the circuit the bias is cut off and gives us the horizontal sync tip

When H2 + H8 + H32 is seen (H42) , HBLANK_DISABLE is turned on, which is a 2nd flop "HSYNC B"
<img width="1032" alt="Screenshot 2025-03-30 at 8 22 18 PM" src="https://github.com/user-attachments/assets/5411d623-2727-4d28-9230-9ce3009e4f3a" />

this lets me chain the hsync events together with explicit start and stop events, block them from re-triggering

<br><br><br><br><br><br><br><br> 

# VBLANK-VSYNC
<img width="1163" alt="Screenshot 2025-03-30 at 7 39 06 PM" src="https://github.com/user-attachments/assets/1bf95850-29ed-48e0-9b7e-5c5c4ec7a497" />

<br><br><br><br><br><br><br><br> 

# COLORBURST
<img width="999" alt="Screenshot 2025-03-30 at 7 39 46 PM" src="https://github.com/user-attachments/assets/0e186959-1d5a-49e6-88f5-805101dba97b" />
<img width="995" alt="Screenshot 2025-03-30 at 7 39 58 PM" src="https://github.com/user-attachments/assets/3ae5673a-1448-4435-93e2-b093c737a3ff" />

<br><br><br><br><br><br><br><br> 

# COLORSELECT
<img width="866" alt="Screenshot 2025-03-30 at 7 41 10 PM" src="https://github.com/user-attachments/assets/f3dcf62d-6dff-4e58-a895-3d3ccc4c33c7" />
<img width="489" alt="Screenshot 2025-03-30 at 7 41 23 PM" src="https://github.com/user-attachments/assets/7ea562ee-2509-4853-81d2-3851e8e37521" />
<img width="351" alt="Screenshot 2025-03-30 at 7 41 27 PM" src="https://github.com/user-attachments/assets/796062e5-a800-4fec-b94d-4860a4f0d022" />

the 4-16 mux is often used for memory addressing and is only available in active-low, so in this case im using NOR gates to invert the signal . the MUX outputs are always high-inactive, so all NOR gates except for the active one are getting a positive voltage which turns them all off, exxcept for the active-low received from the MUX, this NOR gate switches on and provides a color clock at specific delays introduced by the 74AHC451's 4-5ns gate propigation delay
<br><br><br><br><br><br><br><br> 

# BRIGHTNESS
<img width="360" alt="Screenshot 2025-03-30 at 7 42 30 PM" src="https://github.com/user-attachments/assets/d80f44d5-a9c5-4462-be33-3ed7f3564532" />
<img width="787" alt="Screenshot 2025-03-30 at 7 42 13 PM" src="https://github.com/user-attachments/assets/7956db9d-2a4a-49c6-b1d6-9bbe558d9f62" />

<br><br><br><br><br><br><br><br> 

# MEMORYANDBUS
<img width="507" alt="Screenshot 2025-03-30 at 7 43 53 PM" src="https://github.com/user-attachments/assets/5229fa18-7066-4fb3-95a5-0d52f375b51b" />
<img width="482" alt="Screenshot 2025-03-30 at 7 43 57 PM" src="https://github.com/user-attachments/assets/c908dd51-0644-4482-9c94-e30e7bac8f52" />
<br><br><br><br><br><br><br><br> 

# outputamp
<img width="734" alt="Screenshot 2025-03-30 at 8 08 37 PM" src="https://github.com/user-attachments/assets/99eb7eaa-1a76-4243-bb60-a9fcd9228f89" />

<br><br><br><br><br><br><br><br> 

# Components
- 74HC74 D-FLOP
- 74HC08 AND
- 74HC02 NOR
- 74hc244 buffers
- 74hc154 4-to-16 mux
- 74ahc541 inv buffers (color delay line)
- CD4040 12 bit counter
- 14.318mhz crystal
- 128x8 sram HM628128DLP-5
- 2n5087 pnp or equivalent
- 2n7000 mosfet
- 7805 regulator


<br><br><br><br><br><br><br><br> 

# simtools

[LTSpice Model of Output Amplifier](./ltspice_sim/video_output_amplifier.asc)

[Digital Logic simulation](./Digital_sim/ntsc_gen.dig)

output amplifier in ltspice, showing peak brightness vs minimum brightness pixels , there is some distortion in the sim. Some gates are switching on/off before others and causing brief voltage spikes, this might be ignorable on the physical circuit ill have to go through the circuit flow to see what the issue is
![image](https://github.com/user-attachments/assets/067c581b-1207-4646-8b0c-51a6ddc3a029)


