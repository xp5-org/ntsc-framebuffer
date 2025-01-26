an incomplete & in-progress NTSC frame buffer using 74xx logic gates, counters, and a 1mbit sram. its not built or tested

i started with the timing circuit from pong and used 4-16 decoders for color & luminosity

need to finish
- memory & buffers, addressing logic
- potentiometers for output amplifier inputs & biasing


color selection
- each 74AHC buffer adds 4-5ns phase delay and gives more than 16 taps to use for color
![Screenshot 2025-01-26 at 10 27 46 AM](https://github.com/user-attachments/assets/79f72b11-0175-4b36-aa43-b01915a3aadb)


Output amplifier
- mosfet takes colorburst 3.58mhz signal high impedience from logic gates output
- pnp to undo signal invert & drive resistor network
- 2 additional mosfets for changing pnp bias, & resistor output network to control luminosity
![Screenshot 2025-01-26 at 10 24 20 AM](https://github.com/user-attachments/assets/2ff6e7a1-84db-4768-99bd-8a09b2099f5d)



output amplifier in ltspice, showing peak brightness vs minimum brightness pixels , there is some distortion in the sim
![image](https://github.com/user-attachments/assets/067c581b-1207-4646-8b0c-51a6ddc3a029)
