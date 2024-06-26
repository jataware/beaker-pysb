# Description
Run pairwise sensitivity analysis and save the output matrices to CSV files.

# Code
```
from pysb.tools.sensitivity_analysis import PairwiseSensitivity, InitialsSensitivity
from pysb.examples.tyson_oscillator import model
import numpy as np
import numpy.testing as npt
from pysb.simulator.scipyode import ScipyOdeSimulator
import os
import tempfile
import shutil

class TestSensitivityAnalysis(object):
    def setUp(self):
        self.tspan = np.linspace(0, 200, 5001)
        self.observable = 'Y3'
        self.savename = 'sens_out_test'
        self.output_dir = tempfile.mkdtemp()
        self.vals = [.8, .9, 1., 1.1, 1.2]
        self.model = model
        self.solver = ScipyOdeSimulator(self.model,
                                        tspan=self.tspan,
                                        integrator='lsoda',
                                        integrator_options={'rtol': 1e-8, 'atol': 1e-8, 'mxstep': 20000})
        self.sens = PairwiseSensitivity(
            solver=self.solver,
            values_to_sample=self.vals,
            objective_function=self.obj_func_cell_cycle,
            observable=self.observable
        )
        self.p_simulated = np.array([
            [0., 0., 0., 0., 0., 5.0301, 2.6027, 0., -2.5118, -4.5758],
            [0., 0., 0., 0., 0., 5.0301, 2.5832, 0., -2.5313, -4.5823],
            [0., 0., 0., 0., 0., 5.0301, 2.5767, 0., -2.5313, -4.5953],
            [0., 0., 0., 0., 0., 5.0301, 2.5767, 0., -2.5313, -4.6082],
            [0., 0., 0., 0., 0., 5.0301, 2.5767, 0., -2.5313, -4.60829],
            [5.0301, 5.0301, 5.0301, 5.0301, 5.0301, 0., 0., 0., 0., 0.],
            [2.6027, 2.5832, 2.5767, 2.5767, 2.5767, 0., 0., 0., 0., 0.],
            [0., 0., 0., 0., 0., 0., 0., 0., 0., 0.],
            [-2.5118, -2.5313, -2.5313, -2.5313, -2.5313, 0., 0., 0., 0., 0.],
            [-4.5758, -4.5823, -4.5953, -4.6082, -4.6082, 0., 0., 0., 0., 0.]
        ])
        self.sens.run()

    def tearDown(self):
        shutil.rmtree(self.output_dir)

    def obj_func_cell_cycle(self, out):
        timestep = self.tspan[:-1]
        y = out[:-1] - out[1:]
        freq = 0
        local_times = []
        prev = y[0]
        for n in range(1, len(y)):
            if y[n] > 0 > prev:
                local_times.append(timestep[n])
                freq += 1
            prev = y[n]
        local_times = np.array(local_times)
        local_freq = np.average(local_times) / len(local_times) * 2

def test_pmatrix_outfile_exists(self):
    self.sens.run(save_name=self.savename,
                  out_dir=self.output_dir)
    assert os.path.exists(os.path.join(
        self.output_dir, '{}_p_matrix.csv'.format(self.savename)
    ))
    assert os.path.exists(os.path.join(
        self.output_dir, '{}_p_prime_matrix.csv'.format(self.savename)

```
