# Physics-5070-Final-Project-FROG
# Background
In the study of ultrafast optics and lasers and there utilization requires a pulse that is on the femtosecond time scale which is used to study ultrafast carrier dynamics and quantum phenomena. The pulse contains frequency and phase information that when used in population dynamics can be interpreted in a physical way. However, as pulses propagate through optical components (lenses, windows, fibers, etc.), they acquire a frequency-dependent phase $$\( \phi(\omega) \)$$. This results in dispersion and chirp, where different frequency components of the pulse are delayed relative to one another. Chirp leads to temporal broadening and distortion of the pulse, which can strongly affect time-resolved measurements occurring on comparable timescales.

Since most optical photodectors only record the intensity $$ I(t) \propto |E(t)|^2$$,  and then the phase of the electric field $$\( E(t) \)$$ is not directly accessible. This leads to a phase retrival problem, where the full optical field cannot be reconstructed from intensity measurements alone. To overcome the pase retevial problem we use Frequency Resoled Optical Gating (FROG) to reconstruct both amplitude and phase of the pulse. 


## Frequency-Resolved Optical Gating (FROG)
FROG is a nonlinear optical technique used to characterize ultrashort laser pulses. Unlike standard autocorrelation, FROG measures a time-frequency spectrogram, allowing for the full reconstruction of the pulse's amplitude and phase.

### Second-Harmonic Generation (SHG) FROG
<img width="690" height="338" alt="image" src="https://github.com/user-attachments/assets/c2955403-58b7-4c13-8c6b-454d5621dca9" />

The setup utilizes a second-harmonic generation(SHG) crystal, which is $\chi^{(2)}$ nonlinearity. A pulse is split into two replicas; one is delayed by a time $\tau$ relative to the other using a variable delay motorized translation stage. Both pulses are focused into an SHG crystal. When the pulses overlap, they produce a signal at the second-harmonic frequency.

The spectrometer measures the intensity of the signal as a function of frequency $\omega$ and delay $\tau$. The measured FROG spectrogram is defined as, 

$$I_{\text{FROG}}(\omega, \tau) = \left| \int_{-\infty}^{\infty} E_{sig}(t, \tau) e^{-i\omega t} dt \right|^2$$

In the specific case of SHG-FROG, the signal field $E_{sig}(t, \tau)$ is the product of the original field and its delayed replica,

$$E_{sig}(t, \tau) = E(t)E(t-\tau)$$

Substituting this into the intensity equation yields,

$$I_{\text{FROG}}(\omega, \tau) = \left| \int_{-\infty}^{\infty} E(t)E(t-\tau) e^{-i\omega t} dt \right|^2$$.

<img width="1866" height="762" alt="image" src="https://github.com/user-attachments/assets/14e8174b-002a-40a8-bed1-4f838cd31bc3" />


The result is a spectrogram where the y-axis defines the wavelength of our SHG pulse and the x-axis is our $$\tau$$. 

We can then take one additional step and reconstruct the autocorreleation of our pulse by integrating over the intesity cross-section, which is defined by $$A_c(\tau) \propto \int_{-\infty}^{\infty} I(t) I(t-\tau) dt$$.

This extracted autocorrelation is highly useful for two reasons. First, he shape of the intensity autocorrelation provides an excellent starting point for the initial electric field guess used in iterative phase-reversal algorithms. Second, the autocorrelation width is directly related to the pulse duration. By identifying the Full-Width at Half Maximum (FWHM) of the autocorrelation trace, we can estimate the temporal width of the laser pulse (typically by dividing the FWHM by a deconvolution factor, such as $1.41$ for Gaussian pulses or $1.54$ for $sech^2$ pulses).

### Retrival Algorithims
## Common retrieval algorithms
After we have measured our FROG spectrogram, we can now apply several common retrival algorithims depending on the noise and complexity of the pulse.
- **Principal Components Generalized Projections Algorithm (PCGPA)**  
  Most widely used for SHG-FROG

- **Generalized Projections (GP)**  
  Simple but can converge slowly, also used in SHG-FROG

- **ePIE-style variants**  
  More robust for noisy or incomplete data

- **Gradient-based optimization**  
  Minimizes global error metric directly, useful for complex pulses such as those from inteferometric FROG. 

---
For our general purposes and for the low noise data that aquired, we will focus on the GP algorithim. 

## Generalized Projections (GP)

<img width="1001" height="626" alt="image" src="https://github.com/user-attachments/assets/4d119bc2-b128-4571-bf76-a8604d11c55c" />

The GP algorithm is an iterative process that alternates between the experimental constraint(i.e. the measured spectrogram) and the physical constraint(i.e the SHG process). For low-noise data, it provides a reliable and mathematically rigorous way to "undo" the integration and extract the complex field .

### The Iterative Loop
Based on the generalized projections flowchart, the algorithm cycles through the following steps to reconstruct the pulse.

1. The process begins with an initial electric field $E(t)$. While one can "start with noise," using a guess based on the FWHM of the intensity autocorrelation provides a much faster path to convergence.
2. The algorithm calculates the current signal field based on the specific nonlinear process and in our case SHG, $$E_{sig}(t, \tau) = E(t)E(t-\tau)$$
3. The signal field is transformed from the time domain to the frequency domain via a Fast Fourier Transform (FFT) with respect to $t$ result in $$\tilde{E}_{sig}(\omega, \tau) = \mathcal{F}\{E_{sig}(t, \tau)\}$$
4. We then apply a constraint on the measured data in the frequency domain. The algorithm takes the calculated spectral field and replaces its magnitude with the square root of the measured FROG trace, while the calculated phase is kept. This results in $$\tilde{E}'_{sig}(\omega, \tau) = \sqrt{I_{FROG}(\omega, \tau)} \exp(i\phi_{calc})$$.
5. The data-corrected field is transformed back into the time domain using an Inverse FFT with respect to $\omega$ to obtain an updated $E'_{sig}(t, \tau)$.
6. We then apply a physical constriant in the time domain. The algorithm finds a new $E(t)$ that minimizes the functional distance (is "as close as possible") to the modified signal field while strictly obeying the nonlinear optical process and selection rules (SHG).

As mentioned before there are physical and experimental constraints. Below is a desciption of why these constraints must be upheld. 
* The physical constraint represents the physics. It ensures the signal field remains physically possible according to the specific nonlinear interaction (e.g., SHG) occurring in the crystal.
* The experimental constraint presents the experimental reality. Since spectrometers are "phase-blind," this step injects the measured intensity while allowing the algorithm to iteratively refine the unknown phase.

### Convergence and the "FROG Error"
The algorithm repeats this loop until the **FROG Error ($G$)** reaches a minimum threshold. This error represents the Root Mean Square (RMS) difference between the measured and reconstructed traces:

$$G = \sqrt{\frac{1}{N_\omega N_\tau} \sum_{\omega, \tau} \left| I_{meas}(\omega, \tau) - I_{recon}(\omega, \tau) \right|^2}$$

For experimental data with low noise, a $G$ value below **1%** (0.01) is generally considered a successful and converged reconstruction.

### Convergence and the "FROG Error"
The algorithm repeats this loop until the **FROG Error ($G$)** reaches a minimum threshold. This error represents the Root Mean Square (RMS) difference between the measured and reconstructed traces:

$$G = \sqrt{\frac{1}{N_\omega N_\tau} \sum_{\omega, \tau} \left| I_{meas}(\omega, \tau) - I_{recon}(\omega, \tau) \right|^2}$$

For experimental data with low noise, a $G$ value below **1%** is generally considered a successful and "converged" reconstruction.
# Data
There are two folders in here with two different experiments.
# FROG Folder
This was the first FROG spectrogram taken from our 1030 nm, 40 W Carbide laser and the one I demonstrated in the presentation.

# Rep_Rate Folder
This folder contains different experiments where we systematically changed the speed of the pulses from 1MHz( 1 μs) to 250 kHz (4 μs) part of the laser and recorded the change in the spectrograms.
What is rather remarkable is that no matter how bad the noise was in these experiments the algorithim always converged to a solution.


### References
* For more information on FROG principles and variations, see [Swamp Optics - FROG](https://www.swampoptics.com/frog.html).
