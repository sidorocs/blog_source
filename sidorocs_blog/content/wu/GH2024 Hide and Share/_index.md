---
title: "Grehack 2024 ctf write-up : Hide and Share"
type: "articles"
date: 2024-11-17 14:30:00
author: "Mathieu"
description: "The solution of the 'Hide and Share challenge' provided during the Grehack 2024 CTF"
layout: "articles"  # Optionnel : définit le layout utilisé pour cette page
---

As suggested in the challenge description and the source script, the `shares.txt` file contains a set of shares constructed using Shamir's secret sharing scheme. However, a subtlety lay in the encoding of the secret.

Indeed, Shamir's secret sharing usually proceeds as follows:

* We define the following parameters:
  * N -> the number of shares
  * k -> a threshold such that k-1 shares are sufficient to reconstruct the secret

* To generate the shares:
  We take N distinct points and evaluate the following polynomial for each point:

  `s + a_1 * x + a_2 * x^2 + ... + a_k * x^k`

  where a_i for i from 1 to k are randomly chosen numbers.

* To recover the secret, we only need k-1 shares to reconstruct the initial polynomial and evaluate it at 0 to recover the secret `s`.

In our case, the degree-0 coefficient is a linear combination of the secret and the a_i's and is specified by the following 'target vector':
`[1, 2, 1, 0, 0]`
This means that the degree-0 coefficient is composed of `s + 2 * a_1 + a_2`.

To recover the secret, we just needed to find the polynomial coefficients and compute the dot product with the target vector:

```py
def recover_secret(shares, target_vector, prime=_PRIME):
    """
    Recover the secret from share points
    (points (x,y) on the polynomial).
    """
    if len(shares) < k:
        raise ValueError("need at least k shares")
    x_s, y_s = zip(*shares)
    
    coeffs = lagrange_interpolation_coeffs(x_s, y_s, prime)
    secret = dot_product(coeffs, target_vector, prime)
    return number.long_to_bytes(secret)
```

The full solution is available in the `solution.py` file.

By running the script, you will obtain the following output: 
`Recovered secret GH{t4rg3t_v3ct0r_c4n_b3_tr1cky}`