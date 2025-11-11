---

title: BayB Parameters
categories: [CTF, Crypto]
tags: [lattice, rlwe, crypto, medium]
image: images/crypto/baby_Parmeters.png
---------------------------

# BayB — Parameter Recovery & Flag Extraction

This write-up describes how I solved the **Baby_paramters** challenge (parameter recovery / small-error RLWE-style problem) and recovered the flag. The solution uses a small brute-force search over sparse secret vectors and a few simple polynomial operations — the full working script is provided and explained.

> **Note:** This write-up was created using the user's `solve.py` script as the implementation backbone.

# 1. Challenge (summary)

We are given the following public parameters and vectors:

* Modulus: `q = 251`
* Polynomial degree: `n = 96`
* Hamming weight (or related parameter): `du = 125` (provided in the challenge)
* Public polynomial `A` (length `n`)
* Target polynomial `t` (length `n`)
* Auxiliary polynomials `u` and `v` (length `n`)

The goal is to recover a small / sparse secret polynomial `s` (in this case the challenge expects a small Hamming-weight secret) and then derive the message bits (flag) from the recovered secret and the given `u, v` values.

# 2. Observations and approach

* The structure resembles an RLWE-style scheme where `t = A * s + error (mod q)` and the error is small (coefficients centered close to 0).
* The secret `s` is known to be very sparse (low Hamming weight). In the provided `solve.py` the author brute-forces all combinations of positions with `hw = 2` (Hamming weight = 2).
* Once a candidate `s` is found that produces small centered error coefficients, the solver computes `m' = v - (s * u)` and maps polynomial coefficients to bits by thresholding them into ranges around `q/4` and `3*q/4`.

This strategy is simple and effective when the secret weight is extremely small (2 in this case) and `n` is modest (96).

# 3. Implementation (what the script does)

The `solve.py` script performs the following steps:

1. Implements basic polynomial arithmetic modulo `q`:

   * `poly_mul(a, b, n, q)` — multiply polynomials (with ring reduction x^n - 1 implemented by trimming higher degrees).
   * `poly_add`, `poly_sub` — element-wise operations modulo `q`.

2. Loads the challenge arrays `A, t, u, v` and parameters `q, n, du`.

3. Brute-forces candidate secret vectors `s` with Hamming weight `hw = 2` using `itertools.combinations` over indices `range(n)`.

4. For each candidate `s`:

   * Compute `computed_t = A * s (mod q)`.
   * Compute the `error = t - computed_t (mod q)` and check that every coefficient is small when centered (i.e., `abs(e if e <= q//2 else e - q) <= 1`).
   * If the error is small, compute `m' = v - (s * u) (mod q)` and produce bit values by thresholding coefficients into ranges near `q/4` and `3*q/4`. The bits are grouped into bytes and formatted as the flag.

# 4. Result

Using this approach the script prints the recovered flag in the format:

```
Flag: Securinets{<bit-groups>}
```

(See the `solve.py` script for the exact printed groups and the recovered flag.)

# 5. Full script

The working script used to solve the challenge is included with the challenge submission as `solve.py` and was used without modification for the write-up. It implements the brute force approach described above.

# 6. Notes & improvements

* Brute-forcing `hw = 2` over `n = 96` is feasible (the script does about `C(96,2) = 4560` checks), but would quickly become infeasible if the secret weight were larger. For larger weights, lattice techniques or meet-in-the-middle optimizations would be necessary.
* The thresholding mapping from polynomial coefficients to bits assumes the encoding maps bit-0 to values near `0` and bit-1 to values near `q/2` (hence the thresholds near `q/4` and `3*q/4`). Always verify the mapping against the challenge specifics.

# 7. Conclusion

This task is a good example of a parameter-recovery challenge for small-weight secrets in an RLWE-like setting. The combined use of simple polynomial arithmetic and a targeted brute-force on the sparse secret yields a fast and reliable recovery for the given parameters.

---

*If you want, I can:*

* attach the exact `A, t, u, v` vectors and full output of the script in the document,
* or adapt the script to try different assumed Hamming weights (e.g., `hw = 1`..`3`) and include those runs and results.

This is the solve.py
```python

import itertools  # For brute-force

q = 251
n = 96
du = 125

# Polynomial operations
def poly_mul(a, b, n, q):
    c = [0] * (2 * n)
    for i in range(n):
        for j in range(n):
            c[i + j] = (c[i + j] + a[i] * b[j]) % q
    return [(c[i] - c[i + n]) % q for i in range(n)]

def poly_add(a, b, n, q):
    return [(a[i] + b[i]) % q for i in range(n)]

def poly_sub(a, b, n, q):
    return [(a[i] - b[i]) % q for i in range(n)]

# Data (paste A, t, u, v here)
A = [163, 28, 6, 189, 70, 62, 57, 35, 188, 26, 173, 189, 228, 139, 22, 151, 108, 8, 7, 23, 55, 59, 129, 154, 6, 143, 50, 183, 166, 179, 139, 107, 56, 114, 150, 71, 207, 222, 1, 194, 206, 40, 178, 108, 87, 71, 39, 55, 245, 195, 86, 26, 23, 97, 24, 91, 216, 88, 154, 67, 206, 11, 186, 117, 137, 31, 249, 236, 96, 20, 141, 75, 212, 160, 158, 226, 220, 92, 147, 49, 180, 17, 11, 169, 58, 197, 74, 20, 218, 59, 221, 25, 97, 71, 116, 162]
t = [196, 132, 81, 105, 47, 144, 25, 245, 39, 239, 221, 219, 98, 74, 144, 195, 156, 195, 40, 69, 22, 205, 45, 29, 163, 87, 87, 195, 42, 229, 243, 172, 218, 220, 216, 204, 35, 239, 40, 216, 85, 35, 32, 83, 141, 232, 214, 211, 171, 117, 209, 169, 113, 148, 87, 97, 129, 161, 229, 82, 227, 191, 35, 116, 93, 95, 94, 113, 122, 97, 93, 168, 73, 29, 190, 19, 105, 196, 209, 180, 105, 82, 143, 87, 138, 224, 204, 26, 23, 128, 228, 147, 0, 214, 152, 169]
u = [137, 129, 125, 232, 125, 79, 18, 201, 18, 63, 123, 2, 128, 233, 229, 88, 192, 153, 177, 42, 9, 172, 86, 117, 11, 8, 95, 211, 111, 242, 93, 164, 95, 101, 98, 68, 247, 104, 0, 84, 207, 125, 3, 61, 212, 128, 246, 30, 157, 131, 64, 65, 151, 209, 65, 21, 203, 188, 57, 229, 71, 21, 236, 2, 6, 127, 70, 121, 71, 151, 177, 40, 121, 63, 126, 249, 147, 142, 66, 234, 141, 19, 217, 184, 50, 115, 62, 111, 245, 33, 43, 166, 2, 72, 149, 217]
v = [42, 170, 158, 186, 171, 212, 82, 129, 143, 33, 116, 241, 23, 217, 201, 84, 180, 184, 156, 134, 84, 212, 3, 170, 247, 206, 249, 59, 202, 82, 233, 106, 163, 175, 53, 100, 131, 165, 57, 13, 215, 132, 211, 45, 111, 167, 230, 212, 235, 99, 92, 119, 113, 51, 117, 133, 117, 232, 2, 65, 186, 213, 196, 86, 138, 44, 130, 78, 245, 125, 197, 199, 248, 42, 50, 13, 171, 225, 26, 80, 170, 215, 104, 106, 5, 84, 26, 123, 75, 86, 11, 168, 117, 217, 34, 131]

# Brute-force (full code)
import itertools
hw = 2
candidates = itertools.combinations(range(n), hw)
for pos in candidates:
    s = [0] * n
    for p in pos:
        s[p] = 1
    computed_t = poly_mul(A, s, n, q)
    error = poly_sub(t, computed_t, n, q)
    # Check errors small (centered)
    small = all(abs(e if e <= q//2 else e - q) <= 1 for e in error)
    if small:
        inner_su = poly_mul(s, u, n, q)
        m_prime = poly_sub(v, inner_su, n, q)
        m_bits = [1 if q/4 <= coeff < 3*q/4 else 0 for coeff in m_prime]
        bitstring = ''.join(map(str, m_bits))
        groups = [bitstring[i:i+8] for i in range(0, len(bitstring), 8)]
        print("Flag: Securinets{" + " ".join(groups) + "}")
        break


```