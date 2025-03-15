# Embedded-Project
## C Code:
<details>
  <summary>Click to Open </summary>
  
```c 
#include <project.h>
#include <stdint.h>

// Global variable to hold ADC output
volatile uint8_t adc_output = 0;
uint8_t sendData;

int main()
{
    CyGlobalIntEnable;

    UART_Start();        // Initialize UART
    Opamp_Start();       // Initialize OpAmp
    ADC_Start();         // Initialize ADC
    AMux_Start();        // Initialize Analog Multiplexer

    for (;;)
    {
        for (uint8_t channel = 0; channel < 7; channel++)  // Cycle through 7 inputs
        {
            AMux_Select(channel);
            ADC_StartConvert();

            if (ADC_IsEndConversion(ADC_WAIT_FOR_RESULT))
            {
                adc_output = (ADC_GetResult16() >> 8) & 0xFF;
                UART_PutChar(channel);
                CyDelay(2);
                UART_PutChar(adc_output);
                CyDelay(10);
            }
        }
    }
}
```

  </details>


  ## Python Code:

<details>
  <summary>Click to Open </summary>

```c
import serial
import time
import numpy as np
import sounddevice as sd

# Configuration for UART
COM_PORT = 'COM14'  # Change this to your COM port if necessary
BAUD_RATE = 9600

# Map note indices to frequencies (e.g., Sa = 240 Hz, Re = 270 Hz, etc.)
NOTE_FREQUENCIES = {
    0: 240,  # Sa
    1: 270,  # Re
    2: 300,  # Ga
    3: 320,  # Ma
    4: 360,  # Pa
    5: 400,  # Dha
    6: 450   # Ni
}

# Function to play a tone at a given frequency for a specified duration
def play_sound(frequency=262, duration=1):
    sample_rate = 44100  # Samples per second
    t = np.linspace(0, duration, int(sample_rate * duration), endpoint=False)
    wave = 0.5 * np.sin(2 * np.pi * frequency * t)  # Generate sine wave
    sd.play(wave, samplerate=sample_rate)
    sd.wait()  # Wait until sound has finished playing

def main():
    # Open serial port
    with serial.Serial(COM_PORT, BAUD_RATE, timeout=1) as ser:
        print(f"Listening on {COM_PORT} at {BAUD_RATE} baud...")
        sound_playing = False  # Flag to indicate if sound is currently playing
        current_note = None    # Store the current note being played
        
        while True:
            try:
                # Read two bytes from the serial port
                data = ser.read(2)  # Read two bytes
                
                if len(data) == 2:  # Ensure we received exactly two bytes
                    note_byte = data[0]  # First byte: note index (0-6)
                    control_byte = data[1]  # Second byte: control value (0-255)
                    
                    print(f"Received Note Byte: {note_byte}, Control Byte: {control_byte}")

                    # Ensure note_byte is valid
                    if note_byte in NOTE_FREQUENCIES:
                        frequency = NOTE_FREQUENCIES[note_byte]  # Get frequency for the note

                        # Check the control byte value to determine play/stop
                        if control_byte < 200:  # Condition to start playing
                            if not sound_playing or current_note != note_byte:  # Avoid re-triggering the same note
                                print(f"Playing note {note_byte} at {frequency} Hz...")
                                play_sound(frequency, duration=1)
                                sound_playing = True
                                current_note = note_byte
                        else:  # Stop playing if the control byte is 150 or below
                            if sound_playing:
                                print("Stopping sound...")
                                sd.stop()
                                sound_playing = False
                                current_note = None

                else:
                    # If no valid data is received, stop any playing sound
                    if sound_playing:
                        print("No data received, stopping sound...")
                        sd.stop()
                        sound_playing = False
                        current_note = None

            except KeyboardInterrupt:
                print("Exiting...")
                break
            
            except Exception as e:
                print(f"Error: {e}")

if __name__ == "__main__":
    main()
```

  </details>
