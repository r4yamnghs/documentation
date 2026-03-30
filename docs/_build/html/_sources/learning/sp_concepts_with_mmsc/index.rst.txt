Signal Processing Concepts with ADALM-MMSC
===============================================================================

.. note::

   This is a work in progress. Raya is working on this docu!

Introduction
-------------------------------------------------------------------------------


Compressed Introduction Here. All scripts that will be used in this tutorial are 
available in my Repository:

FFT Fundamentals
-------------------------------------------------------------------------------

The **Fast Fourier Transform (FFT)** is a numerical method that converts a
sequence of time‑domain samples into their corresponding **frequency‑domain**
representation. Instead of viewing how a signal changes over time, the FFT
reveals which sinusoidal components are present and how strong they are. This
makes it a central tool for analyzing ADC behavior, harmonic content, and noise
performance in mixed‑signal systems.

When capturing data from the **AD4080 evaluation board**, the FFT enables
identification of the dominant input tone, harmonics, and broadband noise
components. It also provides an indirect but accurate way to observe how the
ADC’s sampling frequency affects the spacing of frequency bins and the placement
of spectral peaks.

An FFT produces a fixed number of equally spaced frequency bins. The spacing
between bins—referred to as the **frequency resolution**—is determined by the
sampling rate and the FFT length:

:math:`f = \frac{f_s}{N}`

Where:
- :math:`f_s` is the sampling frequency  
- :math:`N` is the number of samples in the FFT  

The frequency represented by bin :math:`k` is:

:math:`f_k = \frac{k}{N} \cdot f_s`

The bin with the highest magnitude corresponds to the strongest sinusoidal
component in the signal. If the input frequency is known, its location within
the FFT can be used to estimate the sampling frequency:

:math:`f_s = \frac{f_{in} \cdot N}{k}`

These relationships form the basis for all frequency‑domain tools used in this
tutorial, including sampling‑frequency detection, frequency sweeping, and the
AD4080 sink script’s Fourier analysis.

Sinc Folding Concepts
-------------------------------------------------------------------------------

Oversampling ADCs such as the AD4080 use digital decimation filters to reduce
output data rate while suppressing out-of-band noise. These filters are commonly
implemented as sinc (sin(x)/x) filters, which exhibit a repeating frequency
response made up of lobes and nulls.

As the ADC output sample rate is reduced through decimation, signal and noise
energy outside the baseband folds back into the Nyquist band in a predictable
way defined by the sinc filter response. This effect is known as *sinc folding*.
The shape and severity of the folding depend on both the filter type (sinc1,
sinc5, sinc5 + PF) and the oversampling ratio (OSR).

Higher-order sinc filters provide stronger attenuation near DC but introduce
more pronounced spectral shaping. Increasing OSR reduces the effective signal
bandwidth and increases noise suppression, but also moves the folding points
closer together in frequency. Visualizing these effects helps build intuition
for how digital filtering influences noise, bandwidth, and aliasing behavior in
high-resolution ADC systems.

This script demonstrates sinc folding interactively by injecting controlled,
band-limited noise into the AD4080 and observing how the spectrum is shaped by
different digital filter and decimation settings.


Integrated Noise in Oversampled ADCs
------------------------------------------------------------------------------
Oversampling analog‑to‑digital converters (ADCs) reduce in‑band noise by sampling
the input signal at a rate much higher than the Nyquist frequency and then
applying digital decimation filters to lower the output data rate. This digital
filtering stage suppresses out‑of‑band content while shaping the spectrum of the
remaining noise.

In many precision ADCs, decimation filters are implemented as sinc filters, whose
frequency responses feature regularly spaced nulls and passband droop that
depends on filter order. As a result, wideband (white) noise is not passed
uniformly through the converter; instead, its contribution to the output noise
is weighted by the magnitude‑squared sinc response.

The total RMS noise observed at the ADC output therefore depends on the
integration of noise power across frequency, as shaped by the selected sinc
filter and oversampling ratio. By examining how integrated noise and noise
spectral density vary with bandwidth, filter order, and decimation, it is
possible to gain insight into noise shaping, folding behavior, and effective
signal bandwidth in oversampled data converters.


Using the Detect Sample Frequency Script
-------------------------------------------------------------------------------
To run the **sampling frequency detection (Fs Detect) script**, begin by
powering up both the **ADALM2000** and the **MMSC board** equipped with the
**MAX32666FTHR microcontroller**.

Connect the **W1** output from the ADALM2000 to the **negative input (IN–)** on
connector **P17** of the MMSC board. Ensure that all ground connections are
correctly made. Configure the appropriate COM port on the host computer
to establish communication.

Next, run the Fs Detect script using Python. Open **Scopy**, navigate to the
*Signal Generator*, and configure a sine-wave output. The default input
frequency is typically **100 kHz**, but this value may be adjusted as needed.
Start the signal generator to apply the sine-wave input to the ADC.

**Scopy Sample Setup**

.. figure:: scopy_detect.png
   :align: center
   :width: 900


In the script’s graphical interface, enter the input frequency into the provided
text box. Data acquisition is controlled using the following buttons:

- **Run** → Begins continuous accumulation of ADC samples  
- **Stop** → Halts acquisition  
- **Single Shot** → Captures one buffer of data and stops  

The script identifies the FFT bin containing the dominant spectral peak and
calculates the ADC sampling frequency from its location. The calculated value is
compared to the expected sampling frequency reported by the hardware, and the
error percentage is displayed.

**Expected Output**

.. figure:: frequency_det.png
   :align: center
   :width: 900


Adjusting the physical clock on the AD4080 board causes the ADC sampling
frequency to change. This shift is reflected in the FFT output and calculated
sampling frequency in real time, validating clock control through frequency-
domain analysis.



Using the Frequency Sweep Script
-------------------------------------------------------------------------------


To run the **frequency sweep script**, begin by connecting the **ADALM2000** and
the **AD4080 evaluation board** to your computer. Ensure that the correct USB
interfaces and serial communication ports are specified in the script arguments.
The front‑end sampling frequency must match the rotary switch setting on the
AD4080 board—this is typically **40 MHz**, and the script expects this value to
be set correctly before acquisition begins.

Once the hardware is powered and connected, run the script using Python. The
graphical interface will launch automatically, providing user‑configurable input
fields for sweep parameters including:

- **Start frequency**  
- **Stop frequency**  
- **Step size**  
- **OSR (Oversampling Ratio)**  

Additionally, the interface offers radio buttons to select the desired **digital
filter type**, allowing you to compare responses across different filtering
configurations.

Pressing the **“Run sweep”** button initiates the automated frequency sweep. The
script configures the ADALM2000 to generate sine‑wave stimuli at each frequency
step, while the AD4080 captures the ADC response for every point in the sweep.

During operation, the interface displays three primary plots:

1. **Time‑domain waveform** of the most recent acquisition  
2. **Frequency‑domain spectrum**, highlighting the primary signal components  
3. **Filter frequency response**, plotted in decibels  

**Expected Output with default Frequencies, Sinc5 Filter, and OSR = 1024**

.. figure:: filtersweep1.png
   :align: center
   :width: 900

**Expected Filter Frequency Response**

.. figure:: filtersweep2.png
   :align: center
   :width: 850


By modifying parameters such as OSR or filter type, you can observe how the ADC
response changes in real time. For example:

- Increasing **OSR** narrows the effective bandwidth, shifting the cutoff
  frequency lower.  
- Changing **filter types** alters the roll‑off characteristics, steepness, and
  transition‑band behavior.

Together, these real‑time plots provide a clear and intuitive demonstration of
the AD4080’s digital filtering behavior, enabling users to validate theoretical
expectations against measured data across a full frequency sweep.

Using the AD4080 Sink Script
-------------------------------------------------------------------------------
To run the **AD4080 sink script**, begin by connecting the **ADALM2000** and the
**AD4080 evaluation board** to your computer. Ensure that the correct USB and
serial communication ports are available and selected in the script’s
configuration dialog. The front‑end sampling frequency of the AD4080 must match
the rotary switch setting on the evaluation board—this is typically **40 MHz**,
and the script assumes this value when performing frequency‑domain analysis.

Once the hardware is powered and connected, run the script using Python. A
graphical configuration interface will launch automatically, allowing you to
configure key signal and system parameters, including:

- **Fundamental frequency** of the generated signal  
- **Signal amplitude** specified in dBFS  

**Tab to change parameters**

.. figure:: ad4080_sink1.png
   :align: center
   :width: 450

Default harmonic tones and additional noise components are included to generate
a spectrally rich test signal suitable for frequency‑domain evaluation.

After configuration, the script programs the ADALM2000 to generate a coherent
waveform containing the specified fundamental tone, harmonics, and noise
components. The waveform is transmitted continuously from the M2K, while the
AD4080 captures a buffer of ADC samples corresponding to the applied stimulus.

Once data acquisition completes, the script converts raw ADC codes into voltage
units, normalizes the data to full scale, and performs a Fourier analysis on the
captured samples. The resulting plots display the time‑domain waveform and the
frequency‑domain spectrum, highlighting the fundamental tone, harmonics, and
noise floor.

**Expected Outputs**

.. figure:: ad4080_sink2.png
   :align: center
   :width: 900

This script demonstrates the use of the AD4080 as a high‑resolution data sink in
a mixed‑signal measurement setup. By combining controlled signal generation with
FFT‑based analysis, it enables verification of signal fidelity, scaling
assumptions, and frequency‑domain behavior in a repeatable and closely
controlled environment.

Using the Interactive Sinc Folding Script
-------------------------------------------------------------------------------

To run the Sinc Folding Interactive script, begin by connecting the ADALM2000
(M2K) and the AD4080 evaluation board to your computer. When prompted, select the
appropriate serial port for the AD4080 and the correct LibIIO URI for the M2K.

Ensure that the front-end sampling frequency of the AD4080 matches the rotary
switch setting on the evaluation board. The script assumes a nominal input
sampling frequency of 40 MHz, which is divided by the selected decimation ratio
to determine the effective output data rate.

Once the devices are selected, run the script using Python. An interactive
graphical interface launches automatically and allows you to adjust parameters
such as:

- **Decimation ratio (OSR)**
- **Digital filter type (sinc1, sinc5, sinc5 + PF)**
- **Noise bandwidth**
- **Noise spectral density**

During operation, the script programs the ADALM2000 to generate a band-limited
noise signal centered at a selectable frequency. The noise amplitude is
automatically constrained to remain within the M2K output voltage limits. The
AD4080 captures the resulting ADC output while applying the selected digital
filter and decimation configuration.

**Expected Outputs**

.. figure:: sinc_inter.png
   :align: center
   :width: 900


As parameters are adjusted, the displayed frequency-domain data updates in real
time, clearly showing how noise folds into the baseband and how the sinc filter
shapes the spectrum. This interactive behavior makes the impact of digital
filtering and decimation immediately visible and intuitive.


Using the Noise Sweep Script
------------------------------------------------------------------------------

To run the noise bandwidth sweep script, begin by connecting the ADALM2000 (M2K)
and the AD4080 evaluation board to your computer. When prompted, select the
appropriate serial port for the AD4080 and the correct LibIIO URI for the M2K.

Ensure that the front-end sampling frequency of the AD4080 matches the rotary
switch setting on the evaluation board. The script assumes a fixed input
sampling frequency of 40 MHz when calculating filter notch locations, frequency
axes, and noise bandwidth.

Once the devices are selected, run the script using Python. A control window
appears that allows you to select the desired digital filter type and enter the
oversampling ratio (OSR). Pressing the **Start sweep** button initiates the
measurement.

During operation, the script configures the ADALM2000 to generate band-limited
noise whose bandwidth increases incrementally from DC. At each step, the AD4080
captures a block of ADC samples while applying the selected digital filter and
decimation settings. DC offset is removed from the captured data, and the RMS
noise is computed using the standard deviation of the signal.

**Expected Outputs**

.. figure:: noisesweep.png
   :align: center
   :width: 900

The script generates multiple plots, including time-domain ADC data and its
corresponding frequency spectrum at the final sweep point, total integrated
noise as a function of noise bandwidth, and incremental noise spectral density.
Measured results are overlaid with analytical reference curves based on ideal
rectangular noise and sinc-filter responses, allowing direct comparison between
theoretical expectations and measured behavior.

Practical Notes
-------------------------------------------------------------------------------

Frequency sweep analysis provides a convenient method for validating ADC
performance and clock behavior over a range of input frequencies. Unlike
single-frequency measurements, sweep-based FFT analysis reveals trends,
bandwidth limitations, and frequency-dependent behavior while still avoiding
the need for precise time-domain alignment.