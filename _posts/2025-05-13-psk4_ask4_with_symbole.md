---
title: psk4_ask4_with_symbole
categories: [Matlab]
tags: [code, easy,]
---
# 4-PSK/4-ASK Modulation Simulation in MATLAB

This blog post presents a MATLAB script for simulating a 4-PSK (Phase Shift Keying) or 4-ASK (Amplitude Shift Keying) modulation system. The code generates a binary signal, modulates it into a 4-PSK constellation, adds Gaussian noise based on varying Signal-to-Noise Ratio (SNR) levels, demodulates the signal, and calculates the Bit Error Rate (BER). The results are visualized in scatter plots and BER curves.

## MATLAB Code

Below is the MATLAB code for the simulation:

```matlab
clear
clc
rsb=0:2:20;
N=10000;
for k=1:length(rsb)
    x=randn(1,N);
    for w=1:N
        if x(w)<0
           x(w)=0;
        else
           x(w)=1;
        end
    end
    h=1;
    for j=1:2:N-1
        if x(j)==0 && x(j+1)==0
          xm(h)=(sqrt(2)/2+1i*sqrt(2)/2);
        elseif x(j)==0 && x(j+1)==1
          xm(h)=(-sqrt(2)/2+1i*sqrt(2)/2);
        elseif x(j)==1 && x(j+1)==0
          xm(h)=(-sqrt(2)/2-1i*sqrt(2)/2);
        elseif x(j)==1 && x(j+1)==1
          xm(h)=(sqrt(2)/2-1i*sqrt(2)/2);   
        end
        h=h+1;
    end
    figure(1)
    plot(real(xm),imag(xm),"*")
    
    sigma=(1/sqrt(2))*10^(-rsb(k)/40);
    bruit=sigma*(randn(1,length(xm))+1i*randn(1,length(xm)));
    y=xm+bruit;
    figure(2)
    plot(real(y),imag(y),"o")
    m=1;
    for f = 1:length(y)
        if real(y(f))>0 && imag(y(f))>0
            x_demod(m) = 0;
            x_demod(m+1) = 0;
        elseif real(y(f))<0 && imag(y(f))>0
            x_demod(m) = 0;
            x_demod(m+1) = 1;
        elseif real(y(f))<0 && imag(y(f))<0
            x_demod(m) = 1;
            x_demod(m+1) = 0;
        else
            x_demod(m) = 1;
            x_demod(m+1) = 1;
        end
        m=m+2;
    end
    nb=0;
    for h=1:length(x)
        if x(h)~=x_demod(h)
            nb=nb+1;
        end
    end
    teb(k) = nb/length(x);
end
figure(3)
plot(rsb,teb)
xlabel("rsb")
ylabel("teb")
figure(4)
semilogy(rsb,teb)
xlabel("rsb")
ylabel("teb")
```