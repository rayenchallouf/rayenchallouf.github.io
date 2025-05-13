---
title: ask4_avec_porteuse
categories: [Matlab]
tags: [code, easy,]
---
# 4-ASK Modulation with Carrier Simulation in MATLAB

This blog post presents a MATLAB script for simulating a 4-ASK (Amplitude Shift Keying) modulation system with a carrier signal. The code generates a random binary signal, modulates it into a 4-ASK signal by mapping pairs of bits to amplitude levels, multiplies it with a cosine carrier, and visualizes the carrier and modulated signals.

## MATLAB Code

Below is the MATLAB code for the simulation:

```matlab
clear 
clc
N=10;
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
k=1;
%modulation ASK-2
for j=1:2:length(x)-1
    if x(j) == 0 && x(j+1) == 0
        xmod(k)=-3;
    elseif x(j) == 0 && x(j+1)== 1
        xmod(k) =-1;
    elseif x(j) == 1 && x(j+1) == 0
        xmod(k)=+1;
    else 
        xmod(k)=+3;
    end
    k=k+1;
end
for k=1:length(xmod)
    smod(k,:)=xmod(k)*p;
end
s=reshape(smod',1,length(t)*length(xmod));
figure(2);
plot(s);
```