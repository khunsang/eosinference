# eosinference
This package can be used to estimate the equation of state (EOS) of nuclear matter from observations of BNS inspirals with gravitaional wave detectors. The EOS describes the pressure as a function of density $p(\rho)$.

The methods used here described in this [paper](https://arxiv.org/abs/1410.8866), although I have made some improvements and changes to just take the chirp mass as fixed. The methods were originally inspired by this [paper](https://arxiv.org/abs/1005.0811) on measuring the EOS from neutron-star mass and radius observations.

# Tutorial on using eosinference
There is a [tutorial](https://github.com/benjaminlackey/eosinference/blob/master/examples/RunEOSInference.ipynb) that describes the input data format for each BNS, setting the priors, and generating an output html page.  

# Custom parameterized EOS model
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
        return self.mmax

    def max_speed_of_sound(self):
        """Calculate the maximum speed of sound at any density
        up to the central density of the maximum mass NS (units of v/c).
        """
        return self.v_max

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
An EOS object `eos = YourEOSClass(params)` will be instantiated exactly once per iteration of the emcee sampler. If there are expensive intermediate results needed to calculate quantities such as `eos.lambdaofm(m)` or `eos.max_mass()`, or if results are reused for other methods, you should store them in the `eos.__init__(params)` method so you only have to calculate them once per iteration of the sampler.

You will also need a function that finds a reasonable starting parameter for each walker in the emcee chain.
```python
def your_eos_class_initial_walker_sample():
    """Choose EOS parameters for initializing a single emcee walker.
    This should be sampled from a distribution so each walker has different parameters.
    
    params: 1d-array of EOS parameters.
    """    
    returns params
```
