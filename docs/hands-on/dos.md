---
title: Density of States calculation
sidebar_label: DOS calculation
---

Before we can run the Density of States (DOS) calculation, we need

1. Perform fixed-ion self consistent filed (scf) calculation.
2. Fix the parameters and run non-self consistent field (nscf) calculation with
denser k-point grid.

I have created a new input file (`si_scf_dos.in`) which is very much same as our
previous scf input file except some parameters are modified. You can find all
the input files in my [GitHub repository](https://github.com/pranabdas/qe-dft/).
We used the lattice constant value that we obtained from the relaxation
calculation. We should not directly use the experimental/real lattice constant
value. Depending on the method and pseudo-potential, it might result stress in
the system. We have increased the `ecutwfc` to have better precision. We run the
scf calculation:

```bash
pw.x < si_scf_dos.in > si_scf_dos.out
```

Next, we have prepared the input file for the `nscf` calculation. Where is have
added `occupations` in the `&system` card as `tetrahedra` (suitable for DOS
calculation). We have increased the number of k-points to 12 × 12 × 12 with
automatic option. Also specify `nosym = .TRUE.` to avoid generation of
additional k-points in low symmetry cases. `outdir` and `prefix` must be same as
in the `scf` step, some of the inputs and output are read from previous step.

```bash
pw.x bash si_nscf_dos.in > si_nscf_dos.out
```

Now our final step is to calculate the density of states. The DOS input file as
follows:
```bash title="src/silicon/si_dos.in"
&DOS
  prefix='silicon',
  outdir='./tmp/',
  fildos='si_dos.dat',
  emin=-9.0,
  emax=16.0
/
```

We run:
```bash
dos.x < si_dos.in > si_dos.out
```
The DOS data in the `si_dos.dat` file that we specified in our input file. We
can plot the DOS:

```python title="notebooks/silicon-dos.ipynb"
import matplotlib.pyplot as plt
from matplotlib import rcParamsDefault
import numpy as np
%matplotlib inline

# load data
fid = open('../src/silicon/si_dos.dat', 'r')
data = fid.readlines()
fid.close()

energy = []
dos = []
idos = []

for row in range(len(data)):
    data_row = data[row]
    if (data_row[0][0] != '#'):
        data_row = data_row[:-1].split('  ')
        energy.append(float(data_row[1]))
        dos.append(float(data_row[2]))
        idos.append(float(data_row[3]))

energy = np.asarray(energy)
dos = np.asarray(dos)

# make plot
plt.figure(figsize = (12, 6))
plt.plot(energy, dos, linewidth=0.75, color='red')
plt.yticks([])
plt.xlabel('Energy (eV)')
plt.ylabel('DOS')
plt.axvline(x=6.642, linewidth=0.5, color='k', linestyle=(0, (8, 10)))
plt.xlim(-6, 16)
plt.ylim(0, )
plt.fill_between(energy, 0, dos, where=(energy < 6.642), facecolor='red', alpha=0.25)
plt.text(6, 1.7, 'Fermi energy', fontsize= med, rotation=90)
plt.show()
```

![DOS](../../static/img/silicon-dos.png)