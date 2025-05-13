---
title: ask2_avec_porteuse
categories: [Matlab]
tags: [code, easy,]
---
# 2-ASK Modulation with Carrier Simulation in MATLA

This blog post presents a MATLAB script for simulating a 2-ASK (Amplitude Shift Keying) modulation system with a carrier signal. The code generates a random binary signal, modulates it into a 2-ASK signal by mapping bits to amplitude levels, multiplies it with a cosine carrier, and visualizes the carrier and modulated signals.

## MATLAB Code

Below is the MATLAB code for the simulation:

```matlab
clear 
clc
N=5;
t=0:0.01:0.2;
l0=0;
f0=10;
w0=2* pi *f0;
h=1;
p=cos(w0*t+l0);
figure(1);
plot(p);
k=1;
s=0;
%génération des bits d'info
x=randn(1,N);
for j=1:length(x)
    if x(j) < 0 
        x(j)=0;
    else 
        x(j)=1;
    end
end
%modulation ASK-2
for j=1:length(x)
    if x(j) == 0 
        xmod(j)=-1;
    else 
        xmod(j)=1;
    end
end
for k=1:length(xmod)
    smod(k,:)=xmod(k)*p;
end
s=reshape(smod',1,length(t)*length(xmod));
figure(2);
plot(s);
```