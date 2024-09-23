## Analogue Front-End Components
### Integration and Initial Shaping

The first stage of the analogue front end is to integrate the input charge with an active integrator. As established, the input signal must be amplified, and the integrator can achieve this
with RC values less than unity, according to,

$` V_{\text{out}} = -\frac{1}{RC}\int V_{\text{in}}\,dt`$.

The input charge will be proportional to
the energy of the detected particle, hence the integrator’s importance in particle identification.
The choice of capacitance is informed by the total charge injected from the simulated imput pulse. The
current can be obtained by dividing the equation of the simulated input,

$` V_{in}(t) = A\left\{\exp\left(-\frac{t}{\tau_1}\right) - \exp\left(-\frac{t}{\tau_2}\right)\right\}. `$,

with the resistance through which the the pulse will travel after being emitted by the diode. To generate the simulated pulse in reality, a test capacitor will have to discharge through a resistor to create the exponential envelope desired. An arbitrary value resistance is chose here, at 18 Ω, and the simulated pulse is apprxoimately 100 ns in duration as seen in the theory file. The charge injected into the circuit can be estimated by integrating the current over the pulse width, T = 100 ns,

$` Q_{\text{tot}} = \frac{1}{R}\int_{0}^{T} V_{\text{in}}\,dt\approx92\times10^{-15}\, \text{C} `$.

The op-amps and buffers discussed in the desin considerations operate at ±5 V which is therefore the
maximum output magnitude of the integrator. A small voltage drop would be seen such that
the true maximum output is slightly less, hence the capacitor value is chosen from Q/V , with
V slightly below 5 V. This means using a small ~25 ×10−15 F capacitor would be favourable,
with SMD models meeting this criteria. The small capacitance results in a short time constant,
meaning the integration is performed rapidly, creating a step-like output at the integrator.

To set the pulse width after the integrator, a high-pass CR filter follows, isolated by a buffer.
A decay constant is introduced, such that the pulse width is reduced to approximately five time
constants (where one time constant, τ is 320 ns in this design) of the CR filter, assuming that
the integration time is negligible [^1]. The integrator’s capacitor must fully discharge before another pulse from the diode arrives, since any remaining charge will introduce an offset on the integrated pulse and hence the output, providing artificially high voltages, and false particle identification. Placing a resistor in parallel with the integrating capacitor provides a controlled discharge path. It must have sufficiently large resistance such that on the time scale of the input pulse, the decay of the capacitor is negligible, and remains to appear step-like.

For the same reasons discussed in the theory section, the profile of the high-pass filter output is not
practical since the maximum amplitude is sharp and short in duration after introducing a decay
tail to the step input from the integrator. A low-pass filter is therefore introduced, with the
same time constant as the high-pass filter; these stages in addition to their outputs are illustrated below along with their Fourier transforms. 

| ![Schematic of the first three components; the integrator, CR filter and RC filter.](./Images/first_three_new.png) |
|:--:| 
| KiCad schematic design of the first three components; the integrator, CR filter and the RC filter. |

| ![Simulated output voltage signals for the first three stages of the design](./Images/first_three_outputs2.png) |
|:--:| 
| Simulated output voltage signals for the first three stages of the design. The discharge of the integrator is slow, and its output appears step-like on the time scale of the input pulse. The CR filter sets the pulse width. |

| ![Fourier transforms of the outputs of the CR and RC filters](./Images/freq_spectra_cr_rc2.png) |
|:--:| 
| Frequency spectra of the CR and RC filter outputs of the figure above. A larger dominance of lower frequency components is seen after passing through the low-pass, RC filter, broadening the sharp pulse outputted from the CR filter. |

As expected, the CR filter output has high frequency
components due to its sharp nature. These components are attenuated by the RC filter, where
their relative magnitudes drop by several orders at frequencies above 10 MHz, whilst the lower
frequency components are mostly unaffected and hence their more dominant contribution. It
is for these reason that the output becomes more smooth, since the reduced noise bandwidth
in the frequency domain results in an increased pulse width in the time domain from Fourier
theory [^2]. This further results in high frequency noise mixtures being attenuated early in the
circuit before propagating to the final output.

## Multiple Low-Pass Filters
The signal-to-noise ratio (SNR) at the output of the RC filter can be further improved with
multiple low-pass filters in series with equal time constants, τ [^3][^4]. The choice of having
equal τ across all filters is not only for simplicity, but is also for symmetrical shaping, where
the output after each subsequent RC filter becomes more Gaussian, as demonstrated below.
It is this waveform that has the best SNR. The number of low-pass filters following the
high-pass filter has a direct effect on the length of the pulse, as the peaking time, the time taken
to reach the maximum amplitude, is given by the number of RC filters, multiplied by the time
constant, set at 320 ns. Simulations demonstrate that using more than four RC filters yields
diminishing returns since the form of the output is negligibly different from a true Gaussian at
this point, and the noise bandwidth reduction significantly slows down, as seen in the Bode plots below.


| ![Output signal after repeating RC sections becoming more Gaussian.](./Images/RC-n.png) |
|:--:| 
| Output signals of $` (RC)^n `$ for the CR input, with improved SNR with increasing n. The sharp peak input is smoothened as expected from Fourier theory. |

| ![Output signal after repeating RC in the frequency domain.](./Images/bode_nrc2.png) |
|:--:| 
| Bode plots for $`n`$-RC filters, showing little change in frequency bandwidth between the third and fourth filter, with the latter curves almost entirely superimposed. |

The peaking time, from transient simulation data, was found to match closely with the
expected times for different n, with the largest deviation seen being 20 ns from the expected
320 ns after one RC filter. Since no trend was found in the slight deviations between expected
peaking times after n filters, the small discrepancies have been attributed to sampling intervals
in KiCad; it is likely that the true peaking times lie between two time steps.
Overall, one sees predictable peaking times, and how the pulse width can be increased to
comply with the parameters of the digitisation components. The pulse width of the output
from the fourth low-pass filter is estimated to be 5.60 ± 0.05 μs. If perfectly Gaussian, the
peaking time would be half the pulse width at 2.8 μs, which is larger than observed. This
further illustrates a slight deviation from a Gaussian waveform, however, it is suggested from
extended simulations that any further filtering would be more detrimental due to larger losses
in pulse height, increasing later amplification requirements.

## Inversion and Offset Removal

Since the integrator at the first stage is inverting, this must be corrected by an inverting op-
amp. Its optimum placement is at the end of the pulse shaping circuit, for best signal output,
since the frequency response of the LMH6611 diminishes with increasing closed-loop gain [^5].
After passing through multiple low-pass filters, the main frequency composition will now be on
the order of hundreds of kHz to MHz, as illustrated below.

| ![Frequency spectrum of the signal after four RC filters.](./Images/.png) |
|:--:| 
| Frequency spectrum of the signal after four RC filters. |


In an inverting configuration, the LMH6611 has a 0-dB bandwidth around 100 MHz for closed loop gains between -2 and -5.
The output signal from the low-pass filters is therefore comfortably within the frequency range
for well-behaved amplification; this would not be true for the signal before the RC filters.
The figure below illustrates an accumulating offset at the output. A summing amplifier is therefore to be introduced after the inverter to
remove any final offsets. The likelihood is that the offset seen in a realised circuit would be
different to that simulated, and hence the choice of resistor values for the summing amplifier
would be based on empirical observations.

| ![Demonstration of offset removal of the output pulse using a summing amplifier.](./Images/final_output.png) |
|:--:| 
| Final output after inversion and offset removal. |

## Footnotes

[^1]: Shenkman AL. Transient analysis of electric power circuits handbook. Springer Science &
Business Media; 2006.

[^2]: Hoffman F. An introduction to Fourier theory. Extraído el. 1997;2.

[^3]: Knoll GF. Radiation detection and measurement. John Wiley & Sons; 2010.

[^4]: Spieler H. Semiconductor detector systems. vol. 12. Oxford university press; 2005.

[^5]: Instruments T. LMH6611/LMH6612 Single Supply 345 MHz Rail-to-Rail Output Am-
plifiers; 2013. Available from: https://www.ti.com/lit/ds/symlink/lmh6611.pdf?ts=
1710012404970.
