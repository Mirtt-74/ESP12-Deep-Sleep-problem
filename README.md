# ESP12-Deep-Sleep-problem

ESP12 deep sleep problem
I encountered a similar issue that others have faced with ESP12 modules. While I’ve previously used ESP12E/F without any problems when waking from deep sleep, I recently bought a new batch of ESP12F modules that don’t wake up correctly—they enter a "zombie mode" instead.

To troubleshoot, I researched various articles on this topic and tried multiple suggestions. Initially, adding a 10kΩ resistor between GPIO7/MISO and the +3.3V pins worked fine. However, this solution is no longer effective.

After a wake-up attempt that resulted in "zombie mode," I used an oscilloscope to analyze the signals. I found a 297.15 kHz square wave signal on the GPIO11/SCLK pin and a similar signal, but in a sawtooth form, on the CS pin. On the GPIO0/FLASH pin, there was a 26 MHz signal oscillating between 1.4V and 2.4V.
Eventually, I discovered the solution. Although I used nicolzo’s program, it alone was insufficient. I also needed to connect a 510Ω resistor between GPIO16 and RST, without using a jumper. After making these changes, the module woke from deep sleep successfully.

To summarize, the following steps helped resolve the problem:
1.	Place a 10kΩ resistor between GPIO7/MISO and the +3.3V pins.
2.	Connect a 510Ω resistor between GPIO16 and RST pins.
3.	Use nicolzo’s program.

***   
The nicolzo’s program:

#define ets_wdt_disable ((void (*)(void))0x400030f0)

#define ets_delay_us ((void (*)(int))0x40002ecc)

#define _R (uint32_t *)0x60000700

void nk_deep_sleep(uint64_t time)

{

  ets_wdt_disable();
  
  *(_R + 4) = 0;
  
  *(_R + 17) = 4;
  
  *(_R + 1) = *(_R + 7) + 5;
  
  *(_R + 6) = 8;
  
  *(_R + 2) = 1 << 20;
  
  ets_delay_us(10);
  
  *(_R + 39) = 0x11;
  
  *(_R + 40) = 3;
  
  *(_R) &= 0xFCF;
  
  *(_R + 1) = *(_R + 7) + (45*(time >> 8));
  
  *(_R + 16) = 0x7F;
  
  *(_R + 2) = 1 << 20;
  
  __asm volatile ("waiti 0");
  
}
