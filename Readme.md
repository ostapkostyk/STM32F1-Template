# STM32F1x Template Project

## Contents
1. [Overview](#section-overview)
2. [Details of implementation](#section-features)
3. [Example](#section-example)
4. [Code Generation](#section-generation)
5. [Other notes](#section-other)

## [Overview](id:section-overview)
This project is the template for "Bare metal" hardware with STM32F1 series microcontroller.
It can also be used as the reference to solve typical tasks.
It is generated by STM32CubeMX source code generator and following has been added to make it good starting point for further developments:
- Standard input and output has been implemented to use functions from standard library like printf(), scanf() and so on
- EEPROM emulation (based on ST code) to store settings/info in the internal flash memory
- Efficient static memory allocator/deallocator
- Timer class implements software timers
- LED class implements hardware independent management of LEDs with for configurable states (On, Off, Blink constantly, Blink N-times)
- Hardware Buttons management handles signals from buttons and implements contacts de-bouncing, configurable levels, state detection, events and time measurement

The project is mix of C and C++ code. Most of C-code is autogenerated code. Uses HAL layer and CMSIS from ST.
Eclipse (I use Mars2) with GCC (arm-none-eabi-gcc toolchain) and GDB are used as development environment.
The project initially is generated from included CubeMX configuration file and user-code implemented correctly, so that hardware configuration can be changed by STM32CubeMX code generator and newly generated code doesn't break user code. However some simple actions must be performed prior and after code generation in order to keep C++ language of the project (STM32CubeMX generates C-code only).

## [Details of implementation](#section-features)
- Standard input and output. Uses any of available HUART ports. Non-blocking (uses interrupts to read/write data). Separate and size-configurable Circular buffers are used to store in/out data. 
- EEPROM emulation allows to read/write user-data from/to flash memory. Configurable size and control of wearing of the flash memory
- Memory manager by Eli Bendersky (open-source). Dynamically allocates memory from a fixed pool that is allocated statically at link-time. Internally, the memory manager allocates memory in quantas to minimize pool fragmentation.
- "Timer" C++ class implements Up/Down timers with Set/Reset/Check/Pause/Continue/GetTime functions. Timers are organized as linked list and every new instance of timer class links to the list of timers so then only one static method should be called to make a tick for all created timers
- "LED" class controls LEDs which are typically present in most hardware. Implementation is hardware independent and calls external functions to set High/Low level of the microcontroller's pin. All instances create a linked list and only one call of static method (typically from the main loop) recalculates all LED's. Available methods are:
	- Turn LED ON constantly
	- Turn it OFF constantly
	- Blink periodically with configurable On and Off time.
	- Blink N-times and then turn off (N-times, On, Off time are configurable)
- "Buttons" class controls connected buttons. Implementation is similarly hardware independent (requires one simple external function to read-out microcontroller's input) and creates linked list of instances with one static method which re-calculates all buttons at ones. Functionality:
	- Configurable active level (parameter decides which level corresponds to "pressed" state)
	- De-bouncing implemented by 3 readings with configurable delay (30ms by default)
	- Two simple states (Pressed and released) and two events (Pressed and released)
	- Measures time for pressed state and released state separately which allows to create advanced actions
  
## [Example:](#section-example)
main.cpp (user code) simply demonstrates usage of above functions. One LED turns ON if button is pressed and turns off when it is released. Second LED turns ON when button has been released and remains ON for the duration of time when the button was pressed. Application counts how many time the button has been pressed and stores this value in emulated EEPROM. After restarting, the program reads this value from the EEPROM, and the third LED flashes this number of times, and then turns off (shows the restored from the flash value).
Also application prints some string using standart output stream to demonstrate std out.

## [Code Generation:](#section-generation)
Base of the project is generated by STM32CubeMX configurator as C project, however current project is C/C++.
That means, changes made by CubeMX don't apply to the project automatically and need to be merged manually.
The easiest way is to rename main.cpp to main.c, then generate code by CubeMX and then rename main.c to main.cpp back.
main.cpp in Git repository is adapted main.c generated by CubeMX, so the structure of code is similar.
User code in main.cpp is enclosed between autogenerated comments used by CubeMX to keep it untouched.
When generating new/changed configuration, CubeMX can change HAL drivers which are already in Git repository 
and also will generate/modify main.c file and .h files.
At the end, to regenerate HW configuration and update HAL drivers the only one thing has to be done - renaming of main.c

## [Other notes](#section-other)
- I tried to make this application deterministic and therefore all instances of classes can be created by constructor but cannot be destroyed (no destructor exists) to prevent memory fragmentation and related problems.

- Footprint of the compiled project:
```
   text	   data	    bss	    dec	    hex	filename
  13632	    124	   4276	  18032	    4670
```