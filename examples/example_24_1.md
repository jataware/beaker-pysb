# Description
Demonstration of the Ras-Raf-MEK-ERK kinase cascade model with initial conditions and observables using the PySB library.

# Code
```
from __future__ import print_function
from pysb import *
from pysb.macros import catalyze_state

Model()

Monomer('Ras', ['k'])
Annotation(Ras, 'http://identifiers.org/uniprot/P01116', 'hasPart')
Annotation(Ras, 'http://identifiers.org/uniprot/P01112', 'hasPart')
Annotation(Ras, 'http://identifiers.org/uniprot/P01111', 'hasPart')
Monomer('Raf', ['s', 'k'], {'s': ['u', 'p']})
Annotation(Raf, 'http://identifiers.org/uniprot/P15056', 'hasPart')
Annotation(Raf, 'http://identifiers.org/uniprot/P04049', 'hasPart')
Annotation(Raf, 'http://identifiers.org/uniprot/P10398', 'hasPart')
Monomer('MEK', ['s218', 's222', 'k'], {'s218': ['u', 'p'], 's222': ['u', 'p']})
Annotation(MEK, 'http://identifiers.org/uniprot/Q02750', 'hasPart')
Annotation(MEK, 'http://identifiers.org/uniprot/P36507', 'hasPart')
Monomer('ERK', ['t185', 'y187'], {'t185': ['u', 'p'], 'y187': ['u', 'p']})
Annotation(ERK, 'http://identifiers.org/uniprot/P27361', 'hasPart')
Annotation(ERK, 'http://identifiers.org/uniprot/P28482', 'hasPart')
Monomer('PP2A', ['ppt'])
Annotation(PP2A, 'http://identifiers.org/mesh/24544')
Monomer('MKP', ['ppt'])
Annotation(MKP, 'http://identifiers.org/mesh/24536')

kf_bind = 1e-5
kr_bind = 1e-1
kcat_phos = 1e-1
kcat_dephos = 3e-3

klist_bind = [kf_bind, kr_bind]
klist_phos = klist_bind + [kcat_phos]
klist_dephos = klist_bind + [kcat_dephos]

def mapk_single(kinase, pptase, substrate, site):
    ppt_substrate = substrate()
    if 'k' in ppt_substrate.monomer.sites:
        ppt_substrate = ppt_substrate(k=None)
    components = catalyze_state(kinase, 'k', substrate, site, site, 'u', 'p', klist_phos)
    components |= catalyze_state(pptase, 'ppt', ppt_substrate, site, site, 'p', 'u', klist_dephos)
    return components

def mapk_double(kinase, pptase, substrate, site1, site2):
    components = mapk_single(kinase, pptase, substrate({site2: 'u'}), site1)
    components |= mapk_single(kinase, pptase, substrate({site1: 'p'}), site2)

# Ras-Raf-MEK-ERK kinase cascade
mapk_single(Ras, PP2A, Raf, 's')
mapk_double(Raf(s='p'), PP2A, MEK, 's218', 's222')
mapk_double(MEK(s218='p', s222='p'), MKP, ERK, 't185', 'y187')

Initial(Ras(k=None), Parameter('Ras_0', 6e4))
Initial(Raf(s='u', k=None), Parameter('Raf_0', 7e4))
Initial(MEK(s218='u', s222='u', k=None), Parameter('MEK_0', 3e6))
Initial(ERK(t185='u', y187='u'), Parameter('ERK_0', 7e5))
Initial(PP2A(ppt=None), Parameter('PP2A_0', 2e5))
Initial(MKP(ppt=None), Parameter('MKP_0', 1.7e4))

Observable('ppMEK', MEK(s218='p', s222='p'))
Observable('ppERK', ERK(t185='p', y187='p'))


if __name__ == '__main__':
    print(__doc__, "\n", model)
    print("""
NOTE: This model code is designed to be imported and programatically
manipulated, not executed directly. The above output is merely a

```
