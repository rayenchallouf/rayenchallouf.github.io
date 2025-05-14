---
title: ASK 4 MODULATION PHASE
categories: [Matlab]
tags: [code, easy]
---

## MATLAB Code

Below is the MATLAB code for the simulation:

```matlab
clear
clc
N=10
x=randn(1,N)
for i = 1:length(x)
        if x(i) > 0
            x(i) = 1;
        else
            x(i) = 0;
        end
    end
    x;

    j = 0;
    for i = 1:2:(length(x) - 1)
        j = j + 1;
        if x(i) == 0
            if x(i + 1) == 0
                xmod(j) = -3;
            else
                xmod(j) = -1;
            end
        else
            if x(i + 1) == 0
                xmod(j) = 1;
            else
                xmod(j) = 3;
            end
        end
    end
    xmod;

    phi=0
   f=10
   t=0:0.01:0.2
   p=cos(2*pi*f*t+phi)

   figure(1)
   plot(p)

   for i=1:length(xmod)
      smod(i,:)=xmod(i)*cos(2*pi*f*t+phi)
   end
   s=reshape(smod',1,length(t)*length(xmod))

   figure(2)
   plot(s)
```
