# ESP32 CTCSS/DTMF Decoder Signal Processing Pipeline

## Overview
This document describes the signal processing pipeline implemented in the ESP32 dual-core CTCSS/DTMF decoder. The system uses a dual-core approach with Core 0 handling signal acquisition and processing, while Core 1 handles detection logic and output management.

The project leverages the computational power of the inexpensive ESP32 microcontroller to perform sophisticated digital signal processing tasks that would typically require more expensive dedicated hardware. By repurposing the ESP32 (normally used for WiFi/Bluetooth applications) and utilizing its dual-core architecture, the system achieves professional-grade tone detection capabilities at a fraction of the cost of commercial solutions. Core 0 handles the computationally intensive tasks (ADC sampling, filtering, FFT processing), while Core 1 manages detection logic and output management, creating an efficient parallel processing pipeline.

## Hardware
- ESP32 dual-core microcontroller
- ADC input on pin 34
- Sampling rate: 4000 Hz
- FFT size: 1024 samples

## Signal Processing Pipeline

### Core 0 Tasks
1. **ADC Sampling** (`sampleADC`)
   - Samples analog input at 4000 Hz
   - Fills buffer of 1024 samples

2. **Bandpass Filtering** (`applyBandpassFilter`)
   - Filters input signal to focus on relevant frequency ranges
   - Reduces noise outside the CTCSS (60-260 Hz) and DTMF (697-1633 Hz) bands

3. **FFT Processing** (`performFFT`)
   - Applies windowing function to reduce spectral leakage
   - Computes FFT using ArduinoFFT library
   - Calculates magnitude spectrum
   - Updates noise floor estimation

4. **DTMF Detection** (`processDtmfDetection`)
   - Uses Goertzel algorithm for targeted frequency analysis
   - Detects DTMF tones (697, 770, 852, 941, 1209, 1336, 1477, 1633 Hz)
   - Calculates SNR for each potential tone
   - Implements debouncing for reliable detection

### Core 1 Tasks
1. **CTCSS Detection** (`processCtcssDetection`)
   - Analyzes FFT results in the CTCSS frequency range (60-260 Hz)
   - Identifies peaks and calculates SNR
   - Implements debouncing for reliable detection
   - Uses frequency spread factor (0.7%) to account for slight frequency variations
   - Maintains detection confidence by requiring consecutive matches within spread tolerance

2. **Output Management** (`updateOutputs`)
   - Updates output pins based on detection results
   - Manages serial debug output

3. **Adaptive Threshold** (`updateAdaptiveThresholds`)
   - Dynamically adjusts detection thresholds based on noise conditions
   - Improves reliability in varying signal environments

## Frequency Calibration
The system implements a multipoint calibration approach for accurate frequency measurement:

### Industry Standards
- CTCSS industry standards require frequency accuracy within ±0.5% for reliable operation
- Adjacent CTCSS tones can be as close as 2.5% apart (e.g., 67.0 Hz and 69.3 Hz)
- Commercial radio equipment typically maintains ±0.2% frequency accuracy
- EIA standard RS-220-A defines the 32 standard CTCSS frequencies and their tolerances

### Implementation
1. **Calibration Tables**
   - 5 calibration points for CTCSS (67, 100, 150, 200, 250 Hz)
   - 8 calibration points for DTMF (697, 770, 852, 941, 1209, 1336, 1477, 1633 Hz)

2. **Correction Application**
   - Linear interpolation between calibration points
   - Separate correction factors for CTCSS and DTMF ranges
   - Ensures frequency measurement accuracy within industry standards (0.5%)

### Theoretical Accuracy Limits

#### CTCSS Detection
- **FFT Resolution**: With 1024-point FFT at 4000 Hz sampling rate, the bin width is ~3.9 Hz
- **Frequency Range**: 60-260 Hz spans approximately 51 FFT bins
- **Theoretical Precision**: Using quadratic interpolation between FFT bins can achieve ~0.1 Hz precision
- **Practical Limits**: System can achieve ~0.3% accuracy (e.g., ±0.2 Hz at 67 Hz) after calibration
- **Limiting Factors**: ADC quantization noise, clock stability, and analog input conditioning

#### DTMF Detection
- **Goertzel Algorithm**: Provides targeted frequency analysis with higher precision than general FFT
- **Frequency Range**: 697-1633 Hz requires wider bandwidth but benefits from stronger signals
- **Theoretical Precision**: Can achieve ~0.5 Hz precision at DTMF frequencies
- **Practical Limits**: System typically achieves ±0.3% accuracy across the DTMF range
- **Advantage**: Goertzel algorithm's focused computation provides better SNR for DTMF detection

## Inter-Core Communication
- Uses semaphores for thread-safe data exchange between cores
- Core 0 updates FFT results, Core 1 reads and processes them

## Performance
- Optimized for real-time processing on ESP32
- Maintains consistent detection with minimal false positives
- Achieves industry-standard frequency accuracy through multipoint calibration

## Signal Flow Visualization

To ensure proper alignment, view this chart with a monospace font:

```
+------------+    +---------+                                                  
|  Audio     |    |  ADC    |    +------------------------------------------+ 
|  Input     +----+ Pin 34  +----+  CORE 0                 CORE 1          | 
|            |    |         |    |                                          | 
+------------+    +---------+    |  +--------------+      +------------+   | 
                                 |  | Sample ADC   |      | CTCSS      |   |    +---------+
                                 |  | 4000 Hz      +------+ Detection  +---+----+ Serial  |
                                 |  |              |      |            |   |    | Output  |
                                 |  +------+------+      +-----+------+   |    +---------+
                                 |         |                    |          |              
                                 |  +------v------+      +-----v------+   |    +---------+
                                 |  | Bandpass    |      | Update     |   |    | Digital |
                                 |  | Filter      |      | Outputs    +---+----+ Output  |
                                 |  |             |      |            |   |    | Pins    |
                                 |  +------+------+      +------------+   |    +---------+
                                 |         |                              |              
                                 |  +------v------+      +------------+   |              
+--------------+                 |  | FFT         |      | Adaptive   |   |              
| Frequency    |                 |  | Processing  +------+ Threshold  |   |              
| Calibration  +-----------------+--+ 1024 samples|      | Updates    |   |              
| Tables       |                 |  |             |      |            |   |              
+--------------+                 |  +------+------+      +------------+   |              
                                 |         |                              |              
                                 |  +------v------+                       |              
                                 |  | DTMF        |                       |              
                                 |  | Detection   |                       |              
                                 |  | (Goertzel)  |                       |              
                                 |  +------------+                       |              
                                 |                                          |              
                                 +------------------------------------------+              
                                       Semaphore Synchronization                          
```

Alternative simplified diagram:

```
[Audio Input] → [ADC Pin 34] → [ESP32]
                                  |
                                  ├─[CORE 0]─────────────────────┐
                                  │ │                            │
                                  │ ├→[Sample ADC]               │
                                  │ │     │                      │
                                  │ ├→[Bandpass Filter]          │
                                  │ │     │                      │
                                  │ ├→[FFT Processing]───────────┼───→[CORE 1]
                                  │ │     │                      │      │
                                  │ └→[DTMF Detection]           │      ├→[CTCSS Detection]
                                  │                              │      │
[Frequency Calibration]───────────┘                              │      ├→[Update Outputs]─→[Outputs]
                                                                 │      │
                                                                 │      └→[Adaptive Thresholds]
                                                                 │
                                                                 └──→[Serial Debug]
```
