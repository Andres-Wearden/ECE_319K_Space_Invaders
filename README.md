# ECE_319K_Space_Invaders

<br />

# DESIGN PROBLEM
The component restrictions for this project played an important role in shaping the final design of the game. We were required to use specific hardware components such as external buttons, a slide pot for analog input, and an LCD for graphical display. These components formed the foundation of the game’s user interface, with buttons used for player controls, such as allowing the player to fire a projectile or pause te game, while the slide-pot allowed for smooth controls with the game, like player movement. This requirement introduced the challenge of precise analog to digital conversion and real-time sampling to ensure responsiveness in the game environment. Another important requirement was the need to incorporate graphics and sound, which had to move in response to either user input or times events. These graphics (or sprites) would represent key game elements such as the player’s character or enemies. The game also had to generate sound through a digital to analog converter (DAC) with sound effects triggered by specific game events such as a button press or an enemy death. The game graphics and sounds had to be synchronized with user actions. This required efficient data flow management for displaying sprite data and efficiently sound generate to not interfere with other game operations.

Another one of the central design challenges of this project was managing timing and real-time constraints, which were essential for ensuring the game’s responsiveness to player actions.The system had to handle user actions instantly, updating position of sprites on the LCD or process user inputs such as button presses or slide-pot movements without noticeable delay. Periodic interrupts were responsible for generating sound, requiring us to synchronize audio outputs with visual events. The timing of these interrupts had to be managed to prevent conflicts between multiple processes, such as sprite movement, sound generation, and input handling. Ensuring that all of these operations occurred without affecting the game’s performance required coordination and a manageable software architecture. Failing to meet these real-time requirements would have resulted in laggy controls or desynchronized audio and visuals, impacting the overall performance.

# DESGIN SOLUTION
The design solution required multiple subsystems to communicate efficiently and produce a completed and functional system that maintained playablility within the constraints of the hardware provided. The project consisted of five main components, each acting as an input or output for the game mechanics. The input components included external buttons and a slide potentiometer for the player controls. These buttons served as the main method to perform actions like shooting or setting the settings to start the game, and the slide-pot was used to control the player’s horizontal movement on the screen. The output components included an LCD for display of the sprites, which included the player’s character, enemies, and bullets, with the game logic determining how these elements moved in response to user inputs and timed events. Lastly the microcontroller used was a TM4C123GH6PM, which operated as the brain of the system. The software running on the microcontroller was responsible for managing all the input/output operations, and ensuring the game logic executed smoothly and efficiently.

## Input Components
The input components were responsible for managing all player controls, and consisted of two main components, the buttons and the slide-pot. The buttons were used for discrete actions and wre connected to the GPIO pins on the microcontroller. The slide-pot on the other hand, served as an analog input device that controlled smooth player movemen . It was connected to the ADC of the microcontroller which was used to smooth out control of the player’s character

## The Buttons
The game controls are managed through the microcontroller's Port E pins, specifically PE1-PE3. These pins correspond to the main buttons responsible for starting the game (in either English or Spanish), pausing the game, and shooting at enemies. Unlike the pullup resistors of the microcontroller, the buttons in this game utilize internal pulldown resistors to ensure positive logic. This means that when a switch is not pressed, the digital signal on the pin is low (logical 0), and when pressed, it's high (logical 1). To handle button presses, I've enabled edge-triggered interrupts on PE1-PE3. This ensures that the software responds to hardware when a pin experiences a rising edge (button press). When a button is pressed, the software interrupts and executes specific commands defined within the GPIOPortE_Handler function in the code. For instance, if the software detects that the PE1-connected switch was pressed at the start of the game, the game will play in Spanish. If PE3's switch is pressed at the beginning menu, the game will play in English. During the game, PE3 is used to shoot bullets, and the switch on PE2 is used to pause the game. All these button-related commands are outlined in the GPIOPortE_Handler, which activates each time the software detects a rising edge on one of the enabled pins.

## Slide Potentiometer
In the system, the slidepot is used to control the horizontal movement of the player on the screen. The output voltage from the slidepot ranges from 0V to 3.3V. Depending on the slider's position, this voltage change corresponds to the player's horizontal position on the LCD screen. To make this information usable, the output voltage goes through several processes to be converted into a practical number representing the player's position. The TM4C microcontroller, equipped with 4 analog-to-digital sequencers and 12 analog input channels, is crucial for this process. The slidepot output voltage is connected to port D pin 2, a valid analog input channel on the TM4C. The output voltage, is then converted into a 12-bit digital number through the internal hardware ADC. This digital number spans from 0 to 4095, representing the range of the slidepot output voltage. After obtaining the digital number, the data is sampled through the Timer3A interrupt service routine running at 10 Hz. This ISR continuously samples the 12-bit digital signal for further processing. The sampled data is then converted into a position by inputting the 12-bit number into a function relating the output voltage to the player's horizontal location on the LCD screen. In this case, the function is defined as (Position = (x*2685)/100000), where x is the digital input, and "Position" represents the player's position in terms of pixels away from the left edge of the screen. A digital input of 0 indicates the player is all the way to the left, while a digital input of 4095 indicates the player is all the way to the right of the screen. The vertical position remains constant at y = 160, ensuring the player does not move vertically on the screen.

## Output Components
The output components consisted of the LCD and a speaker responsible for providing real-time feedback to the player. The LCD displayed the game’s visual elements, and was driven by software that refreshed the screen based on user input and timed events. The speaker on the other hand was responsible for generating sound effects that corresponded to actions performed in the game, such as firing a projectile or collisions with enemies. Both of these components worked together to create well-aligned audio and visuals that enhanced the user experience.

## The LCD
I used the ST7735R LCD screen as the output component for displaying the game’s graphics. Since the LCD operated at a much slower speed than the microcontroller, a device driver to control communication between the microcontroller and the display was required to interact with the display. To do this, I implemented functions SPIOutCommand and SPIOutData which handled the communication between the microcontroller and the LCD using busy-wait synchronization. The LCD communicates using the Serial Peripheral Interface (SPI) protocol, with specific pins dedicated to sending commands and data. 

## SPIOutCommand
SPIOutCommand was responsible for sending commands to the LCD, and it would start by checking the if the Busy Bit from the SPI status register bit is high, meaning the SPI interface is busy. If the interface is available, it’ll set the LCD to command mode by clearing bit 6 of the D/C pin, so that the next transmission to the LCD is interpreted as a command and not as data. The data is then sent by writing to the SPI data register. The function again checks the SPI status register to see if the Busy Bit is cleared, indicating that the command has finished transmitting and exits the function. These commands will typically tell the LCD to perform a specific operation, such as clearing the screen or setting some display properties.

## SPIOutData 
The SPIOutData function was responsible for sending data to the LCD. This function worked similarly to the SPIOutCommand, but with a few differences. First the function begins by checking if bit 1 of the SPI status register is high. This will check if the transmit FIFO is ready or not. If the bit is low, the FIFO is full and no data can be transferred, otherwise if the bit is high it’ll continue. Next the function will set the LCD to data mode by setting bit 6 of the D/C pin. Instead of interpreting transmissions as commands, this will allow the LCD to interpret the transmissions as data. The data is then written into the SPI data register, which will begin the transmission of the data to the LCD. Once the data is sent, the function exits completing it’s task. Both of these functions used busy-wait synchronization to ensure a synchronization communication between he microcontroller and the display. This is very important for the proper rendering of visuals on the LCD.

## Speaker
Using a digital-to-analog converter (DAC) along with periodic interrupts, the task was to create the necessary waveforms to drive a speaker and generate sound. The DAC was designed as a binary-weighted DAC with 6-bit resolution connected to pins PB5-0 on the TM4C. This means it can output 64 different discrete levels of analog signals, which was enough to generate the necessary signals for sound. The microcontroller was programmed to output digital signals from an integer array onto its pins connected to the DAC. These signals were updated periodically using timer interrupts set up with the SysTick hardware timer. 

## SysTick
The SysTick timer is a 24-bit down counter, that works by initialzing it by writing the the SysTick reload value, which specified how many clock cycles the timer should count before it interrupt. Once initialized, the timer starts counting down from the reload value. The current value of the decrementing counter is stored in the SysTick current value register. With each clock cycle, the counter is decremented, and when the counter reaches 0, the SysTick timer will trigger an interrupt where the user defined ISR will e executed. The timer will automatically reload the value stored in the reload register, and start counting down again. The interrupts ensure that the waveform data is outputted to the pins is updated at precise intervals, directly controlling the frequency and quality of the sound produced. This method of producing sound allows for complete control over the generated sounds. By creating an integer array with pre-calculated values that represent a waveform, such as a sine wave or custom sound effect, it’s possible to represent any sound as an analog voltage. Additionally, by changing the value of the SysTick reload value can alter frequency of the sound produced, leading to lower or higher pitched sounds. Using this technique, the sound effect from Space Invaders, like bullets shot or death sounds were generating by varying the waveforms stored in different integer arrays. Each array contained the pre-calculated values for specific sounds, with arrays for different sound effects.

# RESULTS
An image of the game system can be seen in Figure 1. The final system resulted in a functional game that integrated all these components into a user experience. The embedded system had three buttons. One button to select the language at the beginning of the game, allowing players to choose between English or Spanish. After the language was selected, the other two buttons were used for controls. One button was used to fire bullets at the enemies, and the other button to pause or resume the game, giving the player control over the game flow. In addition to the buttons, the slide-pot was used for the horizontal movement of the player’s character on the screen. As you can see, there are three wires connected to the slide-pot, and the orange wire was used to transmit data from the slide-pot to the the microcontroller, which was eventually transferred to the LCD for smooth movemen of the playing character. The LCD screen will successfully interpret player movement and display all the graphics it was programmed to display, including the enemies and bullets. Sprites moved across the screen smoothly and the collision detection system was able to accurately determine hits and misses. Finally, the sound system added the final layer of the experience. All the events occurring in the game, such as firing a projectile or destroying an enemy, triggered specific sound effects generated by the DAC, which is the resistive circuit in Figure 1. These sound effects are synchronzied with the visuals, providing immediate audio cues to the player. Together, these components worked to deliver the responsive and functional game experience that met the projects objectives.

 
Figure 1

# CONCLUSION
This project demonstrated the integration of many different embedded system components to create a video game. By using the TM4C microcontroller, the LCD display, slide potentiometer, buttons and sound system, the game delivered a functional user experience. Each component played an important role in creating the game mechanics and user experience, from the player movement and button responses to the synchronzied sound effects and visuals. This project was able to meet the requirements, and provided insights into embedded systems design, including how to manage data flow and hardware-software integration to build a system that meets real-time constraints.
