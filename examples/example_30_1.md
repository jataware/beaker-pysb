# Description
Simulate the bax_pore_sequential model and plot the results

# Code
```
__future__ import print_function
matplotlib.pyplot as plt
numpy import logspace
pysb.simulator import ScipyOdeSimulator

t = logspace(-3, 5) # 1e-3 to 1e5
print("Simulating...")
x = ScipyOdeSimulator(model).run(tspan=t).all

# Plot trajectory of each pore
for i in range(1, max_size + 1):
    observable = 'Bax%d' % i
    # Map pore size to the central 50% of the YlOrBr color map
    color = plt.cm.YlOrBr(float(i) / max_size / 2 + 0.25)
    plt.plot(t, x[observable], c=color, label=observable)
# Plot Smac species
plt.plot(t, x['mSmac'], c='magenta', label='mSmac')
plt.plot(t, x['cSmac'], c='cyan', label='cSmac')

# Expand the limits a bit to show the min/max levels clearly
plt.ylim([-0.01e5, 1.01e5])
# Show time on a log scale 
plt.xscale('log')
plt.legend(loc='upper right')

```
