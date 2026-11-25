Title: Tortuga                                                                                                                                                                                                        
Author: Cameron Emmert (@camemmert)                                                                                                                                                                                    
Description: Tortuga is the avionics board powering my science fair project, a thrust-vector-controlled rocket, and it combines sensors, status indicators (LEDs and a buzzer), and a Teensy 4.1 that runs the calculations needed to keep the rocket stable.                                                                                                                                                                                
Created On: 10/26/2026

10/27/2026: Today I just laid out the basic components including the MOSFET, screw terminal for the e-match connection, and a couple resistors. The bulk of the work is done from my last iteration so I am just building off of it. I plan to add more safety features this time as well including reverse polarity protection and an arm switch for the MOSFET. I had to import the MOSFET symbol from SnapEDA but everything else was included in the KiCAD libraries. 
Hours: 1

10/29/2026: Today I refined the design a bit and fixed a couple of bugs like IMU gnd trace being on the same rail as the servo gnd trace creating noise. I have attached photos of the pyro section and the whole thing at the moment. I also learned more about Schottky diodes and how to use them. 
Hours: 1
<img width="1968" height="1002" alt="image" src="https://github.com/user-attachments/assets/d40cb7a0-eccd-4966-8b21-381ba381d01c" />
<img width="1706" height="1173" alt="image" src="https://github.com/user-attachments/assets/ff189a2b-06a8-4874-b937-2b8a9f136cea" />

11/1/2026: I decided I want to also add a LoRa radio module for communication. I am using an Adafruit RFM95W breakout with 9 pins because of its availability and compatibility with the Arduino framework. This will allow me to digitally check systems before launch. An issue that I ran into was figuring out how to effectively integrate this into the software. I think I will just add another object in the software and make every write to the SD card also write to my laptop from the LoRa. I could just change the SD function to SD & LoRa functions. Then I would also have to write error messages through the LoRa and be able to access the buzzer for status checks when the board is inside the rocket. I had to make a custom schematic symbol because I couldn’t find any on SnapEDA. I attached an image of the module and the custom symbol I made. I also spent some time wiring up the module within the schematic and adding a couple caps.
Hours: 1.5
<img width="568" height="445" alt="image" src="https://github.com/user-attachments/assets/c524c27e-7131-41c2-ae23-7a6af8fee8e4" />
<img width="382" height="612" alt="image" src="https://github.com/user-attachments/assets/adb54ce9-ae17-4e99-a0c5-f83d48fffb90" />
<img width="653" height="356" alt="image" src="https://github.com/user-attachments/assets/1da3b1e8-4d0a-47af-9c77-36d575f006f7" />

11/4/2026:
I think I finished my new schematic. For simplicity, I changed the LoRa module to a simple BLE based HM-10. I don’t really need telemetry, and with BLE I can still easily arm the rocket from 10-30 meters away. I assigned all of the footprints to each symbol. This was a bit tedious and I had to take a minute to research how to find the correct footprints within the KiCAD library and assign them. Next time I will work on starting the layout. 
Hours: 1
<img width="1911" height="1140" alt="image" src="https://github.com/user-attachments/assets/7bea0a3f-01ed-49e2-95a1-a75d58f581eb" />
<img width="703" height="691" alt="image" src="https://github.com/user-attachments/assets/72221d1f-da58-44dc-bdbb-87b6a0807943" />

11/11/2026:

Today was a big day where I worked on the PCB for like 3 hours. I just had to grind getting all the traces connected. I also created a ground plane in the second layer for all of my ground pins. I didn’t finish, but I made a ton of progress. A big issue that is still unresolved is that the necessary 3.3V line has to run past my Teensy 4.1 module, which blocks a bunch of data pins. I might have to redo the layout; I’m not sure yet. 
Hours: 3

11/12/20206:

I finished the traces today. I kept the 3.3V line running past the Teensy, but I shifted it down, which ultimately let me route the wires through. I also organized the traces so they look much cleaner and neater. I also did the via stitching.                                                                                                                                                                                         
Hours: 2
<img width="1221" height="487" alt="image" src="https://github.com/user-attachments/assets/f4c58869-bb42-4919-b94a-0ce4b84688ff" />

11/14/2026:

I fixed some issues with the power lines and had to move a couple of components to do this. The MPU6050 was cut off from power because of some data lines. Everything is looking pretty good, but I still need to make a CAD to double check clearance issues.         

Hours: 1
<img width="1206" height="476" alt="image" src="https://github.com/user-attachments/assets/7190970a-a517-44fd-b411-e8fb4779f0bc" />

11/23/2026:

Today I built the CAD model of my KiCad board. I had to import some components from SnapEDA and the rest were included. I did this mainly for clearance issues regarding the board itself and my AV bay CAD work. Things are looking pretty good for the moment, so I am just doing some last minute review before I send the model away to be printed.                                                                                                                                                                
Hours: 1.5
<img width="621" height="812" alt="image" src="https://github.com/user-attachments/assets/56102575-a8bc-4603-b469-9483ade5a118" />

