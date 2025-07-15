# ESP32 CTCSS/DTMF Decoder and Encoder

## Project Scope
This project aims to develop a cost-effective, high-precision CTCSS and DTMF decoder/encoder device using the ESP32 microcontroller. The device is designed to operate alongside a repeater controller in community repeater systems, providing reliable tone detection and clean regeneration capabilities.

### Primary Objectives
1. **Accurate Detection**: Achieve industry-standard accuracy (±0.5% or better) in CTCSS and DTMF tone detection
2. **Clean Regeneration**: Produce low-distortion CTCSS and DTMF tones for retransmission
3. **Cost Effectiveness**: Utilize inexpensive ESP32 hardware to replace more expensive dedicated hardware
4. **Reliability**: Ensure consistent operation in varying signal conditions with adaptive thresholds
5. **Calibration**: Implement multipoint calibration for precise frequency measurement and generation

### Use Case
The device serves as an auxiliary component for community repeater systems where:
- Incoming signals may have degraded or noisy CTCSS/DTMF tones
- Clean tone regeneration is required for retransmission
- Multiple CTCSS tones need to be detected and regenerated
- DTMF commands need reliable detection for repeater control functions
- Commercial solutions are cost-prohibitive for amateur radio clubs and community organizations

## Key Features

### Detection Capabilities
- **CTCSS Detection**: 60-260 Hz range covering all standard CTCSS tones
- **DTMF Detection**: Full 16-key DTMF keypad support (0-9, A-D, *, #)
- **High Accuracy**: Better than ±0.3% frequency measurement after calibration
- **Noise Immunity**: Adaptive thresholds and SNR-based detection
- **Debouncing**: Multiple consecutive detections required to confirm tone presence

### Encoding Capabilities
- **CTCSS Generation**: Precise sine wave generation for all standard CTCSS tones
- **DTMF Generation**: Accurate dual-tone generation for all 16 DTMF digits
- **Low Distortion**: Clean sine wave output using direct digital synthesis
- **Calibrated Output**: Frequency-corrected tone generation using the same calibration system

### Hardware Utilization
- **Dual-Core Processing**: Core 0 for signal acquisition, Core 1 for detection logic and tone generation
- **ADC Input**: High-speed sampling at 4000 Hz for detection
- **DAC Output**: 8-bit DAC channels for tone generation
- **Digital I/O**: Status indicators and control interfaces

## Signal Processing Pipeline

### Detection Path
1. **Signal Acquisition**: ADC sampling at 4000 Hz
2. **Preprocessing**: Bandpass filtering to isolate relevant frequency ranges
3. **Spectral Analysis**: 
   - 1024-point FFT for CTCSS detection
   - Goertzel algorithm for targeted DTMF detection
4. **Tone Identification**:
   - Peak detection with frequency interpolation
   - SNR calculation against noise floor
   - Frequency correction using multipoint calibration
5. **Decision Logic**:
   - Debouncing with consecutive detection requirements
   - Frequency spread tolerance (0.7% for CTCSS)
   - Adaptive threshold adjustment

### Generation Path
1. **Tone Selection**: Based on detection or external control
2. **Waveform Synthesis**: Direct Digital Synthesis with phase accumulation
3. **Frequency Correction**: Apply calibration factors to ensure accurate output
4. **DAC Output**: Clean sine wave generation at precise frequencies
5. **Amplitude Control**: Configurable output levels for different applications

## Calibration System
- **Multipoint Approach**: 5 points for CTCSS, 8 points for DTMF
- **User-Friendly Interface**: Simple input of measured frequencies
- **Automatic Calculation**: Correction factors derived from measurements
- **Linear Interpolation**: Smooth correction across the frequency range
- **Bidirectional Application**: Same calibration for both detection and generation

## Performance Specifications
- **CTCSS Frequency Accuracy**: Better than ±0.3% (exceeds industry standard of ±0.5%)
- **DTMF Detection Reliability**: >99% in SNR conditions above 15 dB
- **Processing Latency**: <100ms from input to detection
- **Generation Precision**: Better than 0.1% frequency accuracy
- **Operating Environment**: Designed for continuous operation in repeater sites

## Future Enhancements
- Remote configuration via web interface
- Multiple tone detection for mixed-mode repeaters
- Integration with network-based repeater controllers
- Expanded I/O for direct repeater control integration
- Spectrum analysis and logging capabilities
