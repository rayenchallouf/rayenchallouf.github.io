---
title: ASK 2 RSB CONSTANTE
categories: [Matlab]
tags: [code, easy]
---

## MATLAB Code

Below is the MATLAB code for the simulation:

```matlab
N=100
x=randn(1,N)
RSB=0:2:20
for k=1:length(RSB)
    for i=1:length(x)
      if x(i)>0
        x(i)=1;
      else
        x(i)=0;
      end
    end
    x

    for i=1:length(y)
      if x(i)==0
        xmod(i)=-1;
      else
        xmod(i)=1;
      end
    end
    xmod
    plot(real(xmod),imag(xmod),'o')
    sigma=(1/sqrt(2))*(10^(-RSB(k)/40))
    bruit=sigma*randn(1,N)
    y=xmod+bruit
    plot(real(y),imag(y),'+')

    for i=1:length(xmod)
      if y(i)>0
        xdemod(i)=1;
      else
        xdemod(i)=0;
      end
    end
    xdemod
    nb=0
    for i=1:length(x)
      if x(i)~=xdemod(i)
        nb=nb+1
      end
    end
    TEF(k)=nb/N
    TEF(k)
end
figure1
plot(RSB,TEF)
figure2
semilogy(RSB,TEB)
```
