---
title: codeur_decodeur_ask2
categories: [Matlab]
tags: [code, easy,]
---
# 2-ASK Modulation Simulation in MATLAB

This blog post presents a MATLAB script for simulating a 2-ASK (Amplitude Shift Keying) modulation system. The code generates a random binary signal, computes its parity, and modulates it into a 2-ASK signal by mapping bits to amplitude levels.

## MATLAB Code

Below is the MATLAB code for the simulation:

```matlab
clear
clc
n=10;
rsb=10;
x=randn(1,n);
for j= 1:length(x)
    if x(j) < 0 
        x(j)=0;
    else 
        x(j)=1;
    end
end
for k=1:length(x)
    if mod(x(k),2)==0
        par(k)=0;
    else
        par(k)=1;
    end
end
for j= 1:length(par)
    if x(j) == 0 
        xmod(j)=-1;
    else 
        xmod(j)=1;
    end
end
```