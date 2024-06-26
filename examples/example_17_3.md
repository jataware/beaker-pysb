# Description
Define important catalytic and inhibitory processes for the model.

# Code
```
from pysb import *

Model()
Annotation(model, 'http://identifiers.org/biomodels.db/BIOMD0000000220', 'is')
Annotation(model, 'http://identifiers.org/doi/10.1371/journal.pbio.0060299', 'isDescribedBy')

# Example function definitions

def catalyze(enz, sub, prod, kf, kr, kc):
    """2-step catalytic process"""
    r1_name = 'bind_%s_%s' % (sub.name, enz.name)
    r2_name = 'produce_%s_via_%s' % (prod.name, enz.name)
    E = enz(b=None)
    S = sub(b=None)
    ES = enz(b=1) % sub(b=1)
    P = prod(b=None)
    Rule(r1_name, E + S | ES, kf, kr)
    Rule(r2_name, ES >> E + P, kc)

def catalyze_convert(s1, s2, p, kf, kr, kc):
    """2-step catalytic-type process, but the "catalyst" is effectively consumed"""
    r1_name = 'bind_%s_%s' % (s1.name, s2.name)
    r2_name = 'produce_%s' % p.name
    A = s1(b=None)
    B = s2(b=None)
    AB = s1(b=1) % s2(b=1)
    C = p(b=None)
    Rule(r1_name, A + B | AB, kf, kr)
    Rule(r2_name, AB >> C, kc)

def inhibit(targ, inh, kf, kr):
    """inhibition by complexation/sequestration"""
    r_name = 'inhibit_%s_by_%s' % (targ.name, inh.name)
    T = targ(b=None)
    I = inh(b=None)
    TI = targ(b=1) % inh(b=1)

# L + pR <--> L:pR --> DISC
Parameter('kf1', 4e-07)
Parameter('kr1', 1e-03)
Parameter('kc1', 1e-05)
catalyze_convert(L, pR, DISC, kf1, kr1, kc1)

# flip + DISC <-->  flip:DISC  
Parameter('kf2', 1e-06)
Parameter('kr2', 1e-03)
inhibit(DISC, flip, kf2, kr2)

# pC8 + DISC <--> DISC:pC8 --> C8 + DISC
Parameter('kf3', 1e-06)
Parameter('kr3', 1e-03)
Parameter('kc3', 1e+00)
catalyze(DISC, pC8, C8, kf3, kr3, kc3)

# C8 + BAR <--> BAR:C8 
Parameter('kf4', 1e-06)
Parameter('kr4', 1e-03)
inhibit(BAR, C8, kf4, kr4)

# pC3 + C8 <--> pC3:C8 --> C3 + C8
Parameter('kf5', 1e-07)
Parameter('kr5', 1e-03)
Parameter('kc5', 1e+00)
catalyze(C8, pC3, C3, kf5, kr5, kc5)

# pC6 + C3 <--> pC6:C3 --> C6 + C3
Parameter('kf6', 1e-06)
Parameter('kr6', 1e-03)
Parameter('kc6', 1e+00)
catalyze(C3, pC6, C6, kf6, kr6, kc6)

# pC8 + C6 <--> pC8:C6 --> C8 + C6
Parameter('kf7', 3e-08)
Parameter('kr7', 1e-03)
Parameter('kc7', 1e+00)
catalyze(C6, pC8, C8, kf7, kr7, kc7)

# XIAP + C3 <--> XIAP:C3 --> XIAP + C3_U
Parameter('kf8', 2e-06)
Parameter('kr8', 1e-03)
Parameter('kc8', 1e-01)
catalyze(XIAP, C3, C3_U, kf8, kr8, kc8)

# PARP + C3 <--> PARP:C3 --> CPARP + C3
Parameter('kf9', 1e-06)
Parameter('kr9', 1e-02)
Parameter('kc9', 1e+00)
catalyze(C3, PARP, CPARP, kf9, kr9, kc9)

# Bid + C8 <--> Bid:C8 --> tBid + C8
Parameter('kf10', 1e-07)
Parameter('kr10', 1e-03)
Parameter('kc10', 1e+00)
catalyze(C8, Bid, tBid, kf10, kr10, kc10)

# tBid + Bcl2c <-->  tBid:Bcl2c  
Parameter('kf11', 1e-06)
Parameter('kr11', 1e-03)
inhibit(tBid, Bcl2c, kf11, kr11)

# Bax + tBid <--> Bax:tBid --> aBax + tBid 
Parameter('kf12', 1e-07)
Parameter('kr12', 1e-03)
Parameter('kc12', 1e+00)
catalyze(tBid, Bax, aBax, kf12, kr12, kc12)

# aBax <-->  MBax 
Parameter('kf13', transloc)
Parameter('kr13', transloc)
Rule('transloc_MBax_aBax', aBax(b=None) | MBax(b=None), kf13, kr13)

# MBax + Bcl2 <-->  MBax:Bcl2  
Parameter('kf14', 1e-06/v)
Parameter('kr14', 1e-03)
inhibit(MBax, Bcl2, kf14, kr14)

# MBax + MBax <-->  Bax2
Parameter('kf15', 1e-06/v*2)
Parameter('kr15', 1e-03)
Rule('dimerize_MBax_to_Bax2', MBax(b=None) + MBax(b=None) | Bax2(b=None), kf15, kr15)

# Bax2 + Bcl2 <-->  Bax2:Bcl2  
Parameter('kf16', 1e-06/v)
Parameter('kr16', 1e-03)
inhibit(Bax2, Bcl2, kf16, kr16)

# Bax2 + Bax2 <-->  Bax4
Parameter('kf17', 1e-06/v*2)
Parameter('kr17', 1e-03)
Rule('dimerize_Bax2_to_Bax4', Bax2(b=None) + Bax2(b=None) | Bax4(b=None), kf17, kr17)

# Bax4 + Bcl2 <-->  Bax4:Bcl2  
Parameter('kf18', 1e-06/v)
Parameter('kr18', 1e-03)
inhibit(Bax4, Bcl2, kf18, kr18)

# Bax4 + Mito <-->  Bax4:Mito -->  AMito  
Parameter('kf19', 1e-06/v)
Parameter('kr19', 1e-03)
Parameter('kc19', 1e+00)
catalyze_convert(Bax4, Mito, AMito, kf19, kr19, kc19)

# AMito + mCytoC <-->  AMito:mCytoC --> AMito + ACytoC  
Parameter('kf20', 2e-06/v)
Parameter('kr20', 1e-03)
Parameter('kc20', 1e+01)
catalyze(AMito, mCytoC, ACytoC, kf20, kr20, kc20)

# AMito + mSmac <-->  AMito:mSmac --> AMito + ASmac  
Parameter('kf21', 2e-06/v)
Parameter('kr21', 1e-03)
Parameter('kc21', 1e+01)
catalyze(AMito, mSmac, ASmac, kf21, kr21, kc21)

# ACytoC <-->  cCytoC
Parameter('kf22', transloc)
Parameter('kr22', transloc)
Rule('transloc_cCytoC_ACytoC', ACytoC(b=None) | cCytoC(b=None), kf22, kr22)

# Apaf + cCytoC <-->  Apaf:cCytoC --> aApaf + cCytoC
Parameter('kf23', 5e-07)
Parameter('kr23', 1e-03)
Parameter('kc23', 1e+00)
catalyze(cCytoC, Apaf, aApaf, kf23, kr23, kc23)

# aApaf + pC9 <-->  Apop
Parameter('kf24', 5e-08)
Parameter('kr24', 1e-03)
Rule('bind_aApaf_pC9_as_Apop', aApaf(b=None) + pC9(b=None) | Apop(b=None), kf24, kr24)

# Apop + pC3 <-->  Apop:pC3 --> Apop + C3
Parameter('kf25', 5e-09)
Parameter('kr25', 1e-03)
Parameter('kc25', 1e+00)
catalyze(Apop, pC3, C3, kf25, kr25, kc25)

# ASmac <-->  cSmac
Parameter('kf26', transloc)
Parameter('kr26', transloc)
Rule('transloc_cSmac_ASmac', ASmac(b=None) | cSmac(b=None), kf26, kr26)

# Apop + XIAP <-->  Apop:XIAP  
Parameter('kf27', 2e-06)
Parameter('kr27', 1e-03)
inhibit(Apop, XIAP, kf27, kr27)

# cSmac + XIAP <-->  cSmac:XIAP  
Parameter('kf28', 7e-06)
Parameter('kr28', 1e-03)

```
