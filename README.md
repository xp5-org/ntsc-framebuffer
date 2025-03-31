NTSC framebuffer built around 74x cmos logic gates and an 128k x 8bit sram module 

- 8 bits for color + brightness
- 17 bits address
- transfer data in during vblank period 


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

# hvcounter
This is the core of the circuit, its two 4040 counters, one driving the other. the first counts out 455 which is the number of horizontal pixels per row. the output is binary, and with enough AND gates alongside numerous D type flipflops, specific events can be timed out and set to form the sync timings needed

<img width="1116" alt="hv_counter_schematicblock" src="https://github.com/user-attachments/assets/ac2bd6a3-3dfa-4c98-8c49-ba28c4ea0735" />

# HRESET
<img width="1052" alt="Screenshot 2025-03-30 at 7 25 06 PM" src="https://github.com/user-attachments/assets/34855e6e-d72b-4c5e-9eff-090c872445bc" />
256 + 128 + 64 + 4 + 2
counts out 454 horizontal & sets HRESET flag to high 5V
and on the next clock pulse from not the timer we are resetting but the actual clock source

this could have an additional AND gate to wait for positive cycle of H1 for 256 + 128 + 64 + 4 + 2 + 1 = 455 horizontal pixels instead of 454 

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


# VBLANK-VSYNC
<img width="1163" alt="Screenshot 2025-03-30 at 7 39 06 PM" src="https://github.com/user-attachments/assets/1bf95850-29ed-48e0-9b7e-5c5c4ec7a497" />


# COLORBURST
<img width="999" alt="Screenshot 2025-03-30 at 7 39 46 PM" src="https://github.com/user-attachments/assets/0e186959-1d5a-49e6-88f5-805101dba97b" />
<img width="995" alt="Screenshot 2025-03-30 at 7 39 58 PM" src="https://github.com/user-attachments/assets/3ae5673a-1448-4435-93e2-b093c737a3ff" />


# COLORSELECT
<img width="866" alt="Screenshot 2025-03-30 at 7 41 10 PM" src="https://github.com/user-attachments/assets/f3dcf62d-6dff-4e58-a895-3d3ccc4c33c7" />
<img width="489" alt="Screenshot 2025-03-30 at 7 41 23 PM" src="https://github.com/user-attachments/assets/7ea562ee-2509-4853-81d2-3851e8e37521" />
<img width="351" alt="Screenshot 2025-03-30 at 7 41 27 PM" src="https://github.com/user-attachments/assets/796062e5-a800-4fec-b94d-4860a4f0d022" />

the 4-16 mux is often used for memory addressing and is only available in active-low, so in this case im using NOR gates to invert the signal (I think these were meant to be nand?) 

# BRIGHTNESS
<img width="360" alt="Screenshot 2025-03-30 at 7 42 30 PM" src="https://github.com/user-attachments/assets/d80f44d5-a9c5-4462-be33-3ed7f3564532" />
<img width="787" alt="Screenshot 2025-03-30 at 7 42 13 PM" src="https://github.com/user-attachments/assets/7956db9d-2a4a-49c6-b1d6-9bbe558d9f62" />


# MEMORYANDBUS
<img width="507" alt="Screenshot 2025-03-30 at 7 43 53 PM" src="https://github.com/user-attachments/assets/5229fa18-7066-4fb3-95a5-0d52f375b51b" />
<img width="482" alt="Screenshot 2025-03-30 at 7 43 57 PM" src="https://github.com/user-attachments/assets/c908dd51-0644-4482-9c94-e30e7bac8f52" />

# outputamp
<img width="734" alt="Screenshot 2025-03-30 at 8 08 37 PM" src="https://github.com/user-attachments/assets/99eb7eaa-1a76-4243-bb60-a9fcd9228f89" />


# Components
74HC74 D-FLOP
74HC08 AND
74HC02 NOR
74hc244 buffers 
74hc154 4-to-16 mux
74ahc541 inv buffers (color delay line)
CD4040 12 bit counter
14.318mhz crystal
128x8 sram HM628128DLP-5
2n5087 pnp or equivalent
2n7000 mosfet
7805 regulator


# simtools

output amplifier in ltspice, showing peak brightness vs minimum brightness pixels , there is some distortion in the sim
![image](https://github.com/user-attachments/assets/067c581b-1207-4646-8b0c-51a6ddc3a029)
