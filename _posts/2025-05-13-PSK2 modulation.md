---
title: PSK2 modulation
categories: [Matlab]
tags: [code, easy]
---

## MATLAB Code

Below is the MATLAB code for the simulation:

```matlab

clear
clc
N=2
x=randn(1,N)
for i=1:length(x)
    if x(i)>0
        x(i)=1;
    else
        x(i)=0;
    end
end
x

for i=1:length(x)
    if x(i)==0
        xmod(i)=-1;
    else
        xmod(i)=1;
    end
end
xmod

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
