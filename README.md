# pychi

A Python package for simulating the propagation of optical pulses in nonlinear materials.

To install the package, run
```
pip install pychi
```

The documentation is available under
https://pychi.readthedocs.io/en/latest/


Here is a typical example of the use of pychi to simulate the propagation of a short optical pulse in a nonlinear waveguide exhibiting both cubic and quadratic nonlinearities.

```python
# -*- coding: utf-8 -*-
"""
Created on Mon Feb 28 15:31:47 2022
The waveguide/fiber parameters are first provided, and a Waveguide instance
is created. Then, the pulse parameters are used to create a Light object.
A physical model is then chosen, taking into account different nonlinear
interactions based on the user choice. Finally, a solver is instantiated
and computes the propagation of the pulse in the waveguide with the chosen
nonlinear interactions.
@author: voumardt
"""
import matplotlib.pyplot as plt
import numpy as np
from scipy.constants import c

import pychi


"""
User parameters
"""
### Simulation
t_pts = 2**15

### Light
pulse_duration = 100e-15
pulse_wavelength = 1.56e-06
pulse_energy = 1e-9

### Waveguide
wg_length = 0.001
wg_chi_2 = 1.1e-12
wg_chi_3 = 3.4e-21
wg_a_eff = 1e-12
wg_freq, wg_n_eff = np.load('effective_index.npy')
# wg_n_eff is the effective dispersion of the waveguide considered, sampled on the grid wg_freq


"""
Nonlinear propagation
"""
### Prepare waveguide
waveguide = pychi.materials.Waveguide(wg_freq, wg_n_eff, wg_chi_2, wg_chi_3,
                                wg_a_eff, wg_length, t_pts=t_pts)
# Additional options:
# One can provide beta coefficients (strongly discouraged) overwriting the refractive
# index using waveguide.set_betas(betas, wavelength)
#
# wg_n_eff can be a 2 dimensional array, with first dimension the wavelength dependence
# and second dimension the z dependence.
#
# chi2 and chi3 can be callables, returning a z dependent value. Alternatively, they
# can be defined as one dimensional arrays describing their z dependence, or
# two dimensional arrays describing their z and frequency dependence.
#
# One can use waveguide.set_gamma(gamma) or waveguide.set_n2(n2) to provide
# nonlinear coefficient or nonlinear refractive index and overwrite chi3.
#
# Check documentation for more options and details.


### Prepare input pulse
pulse = pychi.light.Sech(waveguide, pulse_duration, pulse_energy, pulse_wavelength)
# Other available pulse shapes:
# pulse = pychi.light.Gaussian(waveguide, pulse_duration, pulse_energy, pulse_wavelength)
# pulse = pychi.light.Cw(waveguide, pulse_average_power, pulse_wavelength)
# pulse = pychi.light.Arbitrary(waveguide, pulse_frequency_axis, pulse_electric_field, pulse_energy)


### Prepare model
model = pychi.models.SpmChi2Chi3(waveguide, pulse)
# Other models available:
# model = pychi.models.Spm(waveguide, pulse)
# model = pychi.models.Chi2(waveguide, pulse)
# model = pychi.models.Chi3(waveguide, pulse)
# model = pychi.models.SpmChi2(waveguide, pulse)
# model = pychi.models.SpmChi3(waveguide, pulse)
# model = pychi.models.Chi2Chi3(waveguide, pulse)


### Prepare solver, solve
solver = pychi.solvers.Solver(model)
solver.solve()


"""
Plots
"""
pulse.plot_propagation()
# Results can also be accessed via pulse.z_save, pulse.freq, pulse.spectrum, pulse.waveform
# The refractive index and GVD can be seen with waveguide.plot_refractive_index()


```

Typical propagation results would look as follows:
![grafik](https://user-images.githubusercontent.com/97957751/222731448-ff856390-4325-4f39-8817-f508898e2308.png)

Check the examples folder for some specific cases and validation against experimental data.
