# eosinference
This package estimates the equation of state (EOS) of nuclear matter from observations of BNS inspirals with gravitaional wave detectors. The EOS describes the pressure as a function of density $p(\rho)$. It also estimates NS structure properties such as the Radius(Mass) curve:
![](https://github.com/benjaminlackey/eosinference/blob/master/examples/et9bns_output/radius_bounds.png)

The methods used here are described in this [paper](https://arxiv.org/abs/1410.8866), although I have made some improvements and changes to just take the chirp mass as fixed. The methods were originally inspired by this [paper](https://arxiv.org/abs/1005.0811) on measuring the EOS from neutron-star mass and radius observations.

# Dependencies
To run eosinference, you will have to install the following packages:
 * pandas: `pip install --user pandas`
 * h5py: `pip install --user h5py`
 * emcee (I used version 2.2.1): `pip install --user emcee`
 * corner: `pip install --user corner`
 * [lalsuite](https://git.ligo.org/lscsoft/lalsuite): `pip install --user lalsuite`

# Required input data
The input is a set of MCMC runs for each BNS event. Each run should be a CSV file with samples of the following parameters: ($\mathcal{M}$, $q$, $\tilde\Lambda$). The code makes the assumption that the MCMC runs were done using flat (uniform) priors in $q$ and $\tilde\Lambda$. If the priors were not flat to begin with, you must reweight the posterior and then resample the posterior in such a way that the reweighted posterior corresponds to a flat prior.

I suggest sampling uniformly in the parameters ($\mathcal{M}$, $q$, $\tilde\Lambda$, $\theta = \tan^{-1}(\Lambda_2/\Lambda_1)$), then marginalizing over $\theta$ (i.e. dropping the $\theta$ parameter). Modern BNS waveforms require as input, positive values of ($\Lambda_1$, $\Lambda_2$). There is a simple mapping between the cartesian parameters ($\Lambda_1$, $\Lambda_2$) and the polar-like parameters ($\tilde\Lambda$, $\theta$). You can use \[0, pi/2\] as bounds on the polar angle theta. This guarantees positive values of ($\Lambda_1$, $\Lambda_2$). 

# Output pages and data
The following is an example of the output for 9 BNS systems measured with the 3rd generation Einstein Telescope. Each BNS system had an SNR of ~130. There are 2 output pages: one shows the [quasilikelihood of each BNS systems](https://htmlpreview.github.io/?https://github.com/benjaminlackey/eosinference/blob/master/examples/et9bns_output/pseudolikelihood.html) generated by a 2d bounded KDE, and the other shows the measured [EOS parameters and derived NS structure quantities](https://htmlpreview.github.io/?https://github.com/benjaminlackey/eosinference/blob/master/examples/et9bns_output/eos_output_page.html). 

# Tutorial on using eosinference
There is a [tutorial](https://github.com/benjaminlackey/eosinference/blob/master/examples/RunEOSInference.ipynb) that describes the input data format for each BNS, setting the priors, and generating an output html page. 

# Adding your own parameterized EOS model
You can use your own parameterized EOS by creating a class with the following methods.

```python
class YourEOSClass(object):
    def __init__(self, params):
        """Initialize EOS and calculate things necessary for the other methods.
        
        params: 1d-array of EOS parameters.
        """
        self.store_intermediate_results_here
        
    def max_mass(self):
        """Calculate the maximum mass (M_\odot).
        """
        return m_max

    def max_speed_of_sound(self):
        """Calculate the maximum speed of sound at any density
        up to the central density of the maximum mass NS (units of v/c).
        """
        return cs_max

    def radiusofm(self, m):
        """Radius in km.
        m : mass (M_\odot)
        """
        return r

    def lambdaofm(self, m):
        """Dimensionless tidal deformability.
        m : mass (M_\odot)
        """
        return Lambda

    def outside_bounds(self):
        """Determine if the chosen parameters are inside or outside the 
        boundaries of the parameter space.
        """
        if outside:
            return True
        else:
            return False
```
An EOS object `eos = YourEOSClass(params)` will be instantiated exactly once for each walker per iteration of the emcee sampler. If there are expensive intermediate results needed to calculate a quantity (such as `eos.max_mass()`, `eos.max_speed_of_sound()`, `eos.lambdaofm(m)`), you should store them in the `eos.__init__(params)` method so you can reuse them for calculating other quantities. This way you will only have to calculate them once for each walker per iteration.

You will also need a function that finds reasonable starting parameters for each walker in the emcee chain. The parameters sould be sampled from a distribution so that each walker has different parameters.
```python
def initialize_walker_your_eos_class():
    """Choose EOS parameters for initializing a single emcee walker.
    
    params: 1d-array of EOS parameters.
    """    
    return params
```

# Scaling
The evaluation time is currently dominated by the time to evaluate Lambda(mass) curves by solving the TOV equations (as well as finding the maximum mass and speed of sound at the same time) for a specific set of EOS parameters. This is done once for each walker and for each emcee iteration. Therefore, the run time will be approximately (Time to evaluate Lambda(mass) curve) \* Nwalkers \* Niterations. The run time is relatively insensitive to the number of BNS systems. However, the number of walkers should be at least 2 \* (N(BNS systems) + N(EOS parameters))
