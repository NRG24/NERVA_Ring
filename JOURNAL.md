---
title: "NERVA Ring (finishing)"
github: "https://github.com/NRG24/NERVA_Ring"
description: "NERVA Ring is a smart ring that combines heart-rate tracking with an electrodermal activity sensor to detect nervous system activity. The goal is to help users manage their nervous system and alert them before they experience burnout.\n\nI am a student/athlete who, at one point, did not know how to regulate stress, so I am making this project to help others facing the same issue."
created_at: "2026-11-19"
total_time: "33h 30m"
---

# November 19, 2026: Starting point
<!-- fabricate:entry 3454 -->

![](https://fabricate.hackclub-assets.com/b6e40c331530d6eb84b381dfb7b651b929f0d6f089eaf2b2b37de29f3d8233b8/image-png/Screenshot%202026-11-19%20at%208.24.49%E2%80%AFAM.png)
Here is the current starting point of finishing this project. NERVA Ring has two critical issues: 
1. The GSR/EDA circuit is not sensitive enough, or it is simply not producing accurate measurements. This circuit is running off an operational amplifier that measures the resistance between the two electrodes on your finger. Instead, I will be pivoting to measuring conductance.
2. The power management system is terrible and probably dangerous. Instead, I will use an integrated chip to manage the power.

**Total time spent: 30m**

# November 19, 2026 (entry 2): Finding better parts
<!-- fabricate:entry 3459 -->

The Max30102 sensor is not accurate for smart-ring heart rate tracking because it only has a red LED which is better for SpO2. Instead, I'll use the MAXM86161 which has a red, green, and IR LED. However, because the green led uses more voltage, I'll need to include a 5v boost converter.

![](https://fabricate.hackclub-assets.com/3359980b9e63b0324a9ff91413c32a299b22e6533b318953e77907fef05b83f3/image-png/Screenshot%202026-11-19%20at%209.48.46%E2%80%AFAM.png)

The next part I need to switch out is the microcontroller. Right now, I am using the RAYTAC MDBT42v, which has an integrated antenna, however it is way too large to fit into a ring. It is really difficult to find an alternative that is even smaller, has an integrated antenna, and is similar in power. So, after a while of searching I found the uBlox ANNA B402 00b which is open CPU like the Raytac, and is significantly smaller. The only catch is that I have to design a counterpoise for the integrated antenna. This will be difficult because I am using a polyimide FPC instead of the standard FR4 PCB. Regardless, its smaller size will be very beneficial. 

![](https://fabricate.hackclub-assets.com/283def9139d7bd9a0086024c3b61a32b1d6e3aa82493a5bf23d726bb775881ef/image-png/Screenshot%202026-11-19%20at%2010.24.32%E2%80%AFAM.png)

**Total time spent: 1h 30m**

# November 19, 2026 (entry 3): Work on a new PMIC system
<!-- fabricate:entry 3479 -->

My original PMIC worked by using a MOSFET which could detect when the charger is plugged in which would cut the power to the SYS. There is also two resistor dividers. One's used for controlling the gate of the mosfet and the other was used to tell the MCU what the battery percentage is. There's a protection diode so that in the case of a spark or esd, it wouldnt destroy the components on the load. Before it reaches the system, it goes through an LDO which would convert it to 3.3v, which was usable by the MCU. 

There are so many thing wrong with this design. The worst problem was using an LDO instead of a buck because it burns through way more battery power. Instead of redesigning a new circuit, I will use a PMIC which can handle everything I was trying to do more effeciently in a smaller package size. I will use the bq25120ayfpt by TI because its 2.5mm, BGA, and matches my exact specifications. I will design the circuit according to the datasheet. There are some passive components that I don't need to add because I am comfortable with the 10mA 0.5 charging current.

It uses an inductor for more efficient voltage conversion. It also includes I2C to communicate with the MCU about the charge status and battery level which might be annoying because there are already so many other i2c devices on the bus. Also, there is a thermistor pin avaiable, so we'll see if I want to include one for saftey reasons

![](https://fabricate.hackclub-assets.com/1127370f01ee0fe6e6c9127293bd7269c5f2e6287082e01bfde3292b7d24aef2/image-png/Screenshot%202026-11-19%20at%2011.18.18%E2%80%AFAM.png)

**Total time spent: 2h**

# November 19, 2026 (entry 4): Designed the MCU Schematic
<!-- fabricate:entry 3480 -->

I do not want to use the MDBT42v because of the reasons I listed earlier. So, I will have to learn how to implement the ANNA B402 instead. I think because they both use Nordic Semi. Chips they should be relatively similar to set up. Both modules are programmed through Serial Wire which is convenient because I have a Raspberry Pi debug probe for programming. 
![](https://fabricate.hackclub-assets.com/e9afe6d916a885181efe4d23b09a2e6d1a42e49e956924528797818acb886544/image-png/Screenshot%202026-11-19%20at%201.49.56%E2%80%AFPM.png)
This is the current MCU schematic
Also, during PCB layout I have to adhere to these counterpoise guidlines: 

![](https://fabricate.hackclub-assets.com/50801c14ceb7e174a59bac561ce6ddccde3d6d0122ad9fd3141b73479f4b5695/image-png/Screenshot%202026-11-19%20at%202.01.50%E2%80%AFPM.png)


![](https://fabricate.hackclub-assets.com/70feecf749a09964c6a5dae60f0543878c1f3501fa7168c4104a53b616d669af/image-png/Screenshot%202026-11-19%20at%202.03.04%E2%80%AFPM.png)
Setting up the schematic for the new module is even easier than the RAYTAC. It only needs a few decoupling capacitors, but everything else is integrated. The only difficult part will be the antenna.

**Total time spent: 1h**

# November 20, 2026: Create an Electrodermal Activity Sensor
<!-- fabricate:entry 3519 -->

I was originally planning to copy the GSR circuit from the Grove GSR Sensor which looks like this: 
![](https://fabricate.hackclub-assets.com/cac42b86a2da65a99de6897abb82e8ce6007122c9a66941fc18613a137d6f0a2/image-png/Screenshot%202026-11-19%20at%202.27.15%E2%80%AFPM.png)

The issue with this circuit is that its designed for two fingers instead of one. Its also measuring resistance by creating a resistor divider for 0.5v and amplifying it with the OP amp instead of just measuring conductance. My new schematic will use a multi-stage operational amplifier to measure conductance or resistance, I haven't decided yet. 
I will begin by finding a compatible op amp that matches the specs I need for voltage input.
.....
I will go with the OPA2333P primarly because it is anti-drift and I am able to power it with 1.8v. 

This works well because if I shift system load voltage from 3.3v to 1.8v, I can squeeze extra voltage out of the battery and extend battery life.
heres my circuit: 

![](https://fabricate.hackclub-assets.com/aa92d4e2ec6628281c01efbe4b07adf0c450d22348211b58bf90e4717d1eecc7/image-png/Screenshot%202026-11-20%20at%208.54.50%E2%80%AFAM.png)

This paired with the values of the passives will output a 0.5v signal across the sking and should read it at the right frequency according to the 100nF and 220k resistor

**Total time spent: 2h 30m**

# November 20, 2026 (entry 2): Pi Matching Network footpring
<!-- fabricate:entry 3526 -->

Because the MCU and antenna will be enclosed in a ceramic or plastic ring covered in a resin layer and nearby gold electrodes, it is likely that the antenna will be detuned. To counteract this, I need to put footprints for a pi matching network. I have never done this before but now I know it consists of 3 passives. Two shunts which are usually capacitors and one series which is usually an inductor. I also included a 0ohm jumper just in case I dont want to use the network.

![](https://fabricate.hackclub-assets.com/6a7415b9320743f6d0faa6b3d7489c8272a605551873f4bd5336cc7ee6fcfa11/image-png/Screenshot%202026-11-20%20at%2012.41.23%E2%80%AFPM.png)

**Total time spent: 2h**

# November 21, 2026: Boost Converter for the Green LED
<!-- fabricate:entry 3584 -->

I need to add a boost converter (5v) for the MAXM86161's green LED. I don't think I have a lot of space, though, so I will need to find a boost converter that is very small footprint size and does not require a lot of passives. 

I found the TPS61240YFFR which is BGA. It is ultra-small under 2.5mm and I think I will be able to fit it right next to the PPG sensor in layout. 

![](https://fabricate.hackclub-assets.com/c6e4a174dc5097d41cc81f35ad2b9c810af64fbae9e3bdf12ebe81915d4eb942/image-png/Screenshot%202026-11-21%20at%209.39.44%E2%80%AFAM.png)


Here is how I am going to wire this: I want the converter's input to be directly from the battery, not from the buck converter rail because it likely cannot handle the load. Drawing from the battery is safer. And the enable pin will be connnected to a GPIO (with a pull down). If I hadn't added a pull-down, when the MCU turns off, it will leave the EN pin floating which would leave the boost converter on and it would eat my battery life. 
![](https://fabricate.hackclub-assets.com/83490e626a3ef487c1d5144e7c98a3157ba579f1c26501b4fbcc1f9cdea3af8e/image-png/Screenshot%202026-11-21%20at%209.42.54%E2%80%AFAM.png)

**Total time spent: 2h**

# November 21, 2026 (entry 2): New new Electrodermal Activity Sensor
<!-- fabricate:entry 3632 -->

My latest electrodermal activity sensor probably will not work as I want to, and I think it technically still only measures resistance. Also, the opamp is good, but its package size is too large for my liking, I'd rather do another BGA. 

I am going to use LT Spice to simulate and create a new circuit with the new TLV8542RUGR Op Amp.

![](https://fabricate.hackclub-assets.com/df14ec7572619139561ac56dbeb25094aa3e6ac4ed739d5d97624c02af228d04/image-png/Screenshot%202026-11-21%20at%204.21.37%E2%80%AFPM.png)
![](https://fabricate.hackclub-assets.com/425a51cb104a3908f6c90f5c60292723e5bd3ce0bfdad8c0109d67302b66cb84/image-png/Screenshot%202026-11-21%20at%204.21.41%E2%80%AFPM.png)

This is very precise and I created editable python scripts to validate everything. This new circuit requires yet another GPIO pin to read the analog. Now both electrodes are active. One is driving the current and the other is reading it.

**Total time spent: 2h 30m**

# November 21, 2026 (entry 3): Charging Port
<!-- fabricate:entry 3633 -->

I had to find a way to charge the ring that isnt too bulky, but is also reliable. I cannot just slap a USBC onto the PCB because it wouldn't fit in the ring. So I had to look at other smart rings and research how they solved that problem.

Some rings use wireless inductance charging, but I think it would make it too bulky.
Most rings use a 4mm round magnetic pogo. So I had to look online trying to find one that I don't need a 500 MOQ for. 

![](https://fabricate.hackclub-assets.com/aa87f3cb3ec022bbfe607db4fd370704dae912244f56d77c1f4a77a29159192c/image-png/Screenshot%202026-11-21%20at%204.36.10%E2%80%AFPM.png)

Here is what I will use:

There is no available footprint for it so I have to use the drawings provided to make my own:
![](https://fabricate.hackclub-assets.com/6ba081f7ef6aa64c7246b0e31c8141a888525f907536710adb68ac2a712631d8/image-png/Screenshot%202026-11-21%20at%204.38.13%E2%80%AFPM.png)
This will work: it fits the dimensions and also allows me to route out the center voltage pin.

Time to make the charger connected to the circuit
![](https://fabricate.hackclub-assets.com/fcab6967a5b676bad239a9bae169f143322d2ec5b43e1380841e405549aadc04/image-png/Screenshot%202026-11-21%20at%204.39.01%E2%80%AFPM.png)

Done

**Total time spent: 1h 30m**

# November 21, 2026 (entry 4): Schematic Final Touches
<!-- fabricate:entry 3666 -->

I changed some passive values because when I made the passives mostly 0201 and 0402s so I compensated by raising some of the values of the bulk caps. I also renamed some net labels to make it clearer.

![](https://fabricate.hackclub-assets.com/3871c70774414211cac5677b06fe4e14f3ff39a32d146d7edeb15c947b9cb055/image-png/Screenshot%202026-11-21%20at%208.03.50%E2%80%AFPM.png)

**Total time spent: 1h 30m**

# November 22, 2026: Layout Step 1 Place components.
<!-- fabricate:entry 3709 -->

Layout for this smart ring is a brutal process. I have to keep the number of stiffener islands under 8 so JLCPCB doesn't charge an extra $40. I also have to make sure that the PPG sensor will be in the middle of the board while also keeping each electrode equally far apart at 11mm. 

I wanted the charger to be close to the PMIC chip. I put the battery terminals on the bottom layer supported by a top layer stiffener. 

I redid this layout several times until it was thin enough for my liking. 

![](https://fabricate.hackclub-assets.com/d16324d5734e7ac02631ebf09326dd7c7cf6e1e459b3e7061eb5a4c6bf925d3c/image-png/Screenshot%202026-11-22%20at%2010.57.32%E2%80%AFAM.png)

**Total time spent: 4h**

# November 22, 2026 (entry 2): Antenna Counterpoise
<!-- fabricate:entry 3711 -->

Followed the ANNA B4 Datasheet to design the counterpoise in the corner of the board. This was difficult I had to decide whether to use a copper trace or a copper pour. I went with copper pour. Then with lines, I had to sketch out the dimensions. Then I placed the copper pour outline and also the No-Pour region.

![](https://fabricate.hackclub-assets.com/12a839e7288f22fbbe69d29063cd0b79a1ca9e3776c522d151ff90ec1018d091/image-png/Screenshot%202026-11-22%20at%2010.57.59%E2%80%AFAM.png)

**Total time spent: 2h**

# November 22, 2026 (entry 3): Design the housing in CAD
<!-- fabricate:entry 3727 -->

I need a super thin housing with edge rails to make it easy and reliable to assemble the PCB into it. I have a manufacturer lined up that does 0.4mm min wall thickness so I'll design with that in mind for my edge rails. 

![](https://fabricate.hackclub-assets.com/525b6935f65409239284aed8d25205817403949ad3c7e0d165124e941906b488/image-png/Screenshot%202026-11-22%20at%204.20.07%E2%80%AFPM.png)

I added a notch as a reference point for where the battery should go.

**Total time spent: 1h**

# November 22, 2026 (entry 4): Designing the Electrode Pogo
<!-- fabricate:entry 3730 -->

I need an electrode that doesn't take too much space on the PCB, but also provides enough surface area for an accurate reading on the skin. 
At first I was thinking of a rectangular shaped electrode which would have been very convinient, however I looked into the cost for custom manufacturing that part, and it is way too expensive. It would have looked like this: 

![](https://fabricate.hackclub-assets.com/64f10a4ff482efbcf3ddecf24725eb6abeb67c4f5b1201886c2bcdf6ffb0c869/image-png/Screenshot%202026-08-25%20at%201.16.13%E2%80%AFAM.png)
For that reason I am going to do a 3mm footprint on the PCB where it is a cylinder for a few mm and then expands to a 5mm+ face, sort of like a rivet. Here's my modeled idea. 
![](https://fabricate.hackclub-assets.com/d7d5a953c880c32e3ef4770c845335a6972f77587e123fd043810cc20a410fe0/image-png/Screenshot%202026-11-22%20at%204.34.46%E2%80%AFPM.png)

**Total time spent: 1h 30m**

# November 23, 2026: Routing the PCB
<!-- fabricate:entry 3808 -->

I routed the PCB which was one of the hardest things about this project of all time. I did it by routing the extensive lines first (i2c and 1v8 power lines) and then did each indiviual stiffener island one by one. This was brutal. 

![](https://fabricate.hackclub-assets.com/adaadd3c349949d46c98fdba4ddfd71c0d869f2bbee3ceb5b479c44ab0cedd99/image-png/Screenshot%202026-11-23%20at%209.35.36%E2%80%AFPM.png)
![](https://fabricate.hackclub-assets.com/a57001234ee04b8919173d260c2acfe2bf24f7178e29a972297c75f9e5ff6871/image-png/Screenshot%202026-11-23%20at%209.35.41%E2%80%AFPM.png)

**Total time spent: 8h**
