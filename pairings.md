# Embedding degree

A subgroup G of an elliptic curve E(F_q) is said to have embedding degree k if the subgroup order divides q^k - 1, but does not divide q^i - 1 for 0 < i < k.

To have efficient pairings k needs to be small enough for computations in the field F_(p^k) to be feasible, but also big enough to have a required level of security.

Let E be an elliptic curve over the finite field F_q. Let N be the number of F_q points on E and l be a prime divisor of N. In crypto applications, E is always chosen so that N is divisible by a large prime l. Thus, solving dlog in the cyclic group of l points is hard.

## Menezes-Okamoto-Vanstone (MOV) attack

MOV attack uses Weil pairing to move dlog problem in E(F_q) into F_(q^k). Dlog in finite fields can be attacked by index calculus methods and can be thus solved faster than dlog in elliptic curves.

We are trying to find x from x*P. If we have linearly independent P and Q, we have e(x*P, Q) = e(P, Q)^x where e(P, Q) is not 1. Thus, we can find x by breaking dlog in F_(q^k).

# Types of pairings (see [1])

Pairing is a function e:

```
e: G1 x G2 -> GT
```

where G1, G2, GT are cyclic groups of prime order l. If G1 = G2, we say it is a symmetric pairing.

Pairings can be separated in four different types:

 * G1 = G2
 * G1 != G2, but there is an efficiently computable homomorphism phi: G2 -> G1 (but not from G1 to G2)
 * G1 != G2 and there are no efficiently computable homomorphisms between G1 and G2
 * G2 is a non-cyclic group of order l^2.

Note that the cyclic groups of the same order are isomorphic, so there is an isomorhpism between G1 and G2, but it might not be efficiently computable. 

Let's say q is the size of the field over which elliptic curve E is defined. G1 is a subgroup of E(F_q), G2 is usually a subgroup of E(F_q^k), GT is a subgroup of F_q^k\*. Thus, there are three main parameters: field size q, embedding degree k, group size l.

## Type 1

When G1 = G2, Weil pairing becomes trivial since G1 is a cylic group and two elements are always dependent, let's say we have P, Q, then Q = l*P for some l. Then:

```
e(P, Q) = e(P, l*P) = e(P, P)^l = 1
```

In type 1 pairings we actually use a distortion map: 

```
psi: E(K)[r] -> E[r]\G1.
psi(x,y) = (-x, i*y) where i^2 = -1
```

## Type 2

Let's have a curve E over F_q with embedding degree k > 1. G1 is subgroup of E)F_q) of order l. For G2 we choose a random point from E(F_q^k)[l] and define G2 = <Q>. Q is published as system parameter.

Let's observe two maps - Frobenius map and trace map. Frobenius map is: pi: x -> x^q. Note that Frobenius map is a generator of Galois group Gal(F_q^k, F_q).

Trace map is:

```
Tr: E(F_q^k) -> E(F_q)
Tr(x) = x + pi^1(x) + pi^2(x) + ... + pi^(k-1)(x) = x + x^q + x^q^2 + ... + x^q^(k-1)
```

If we calculate Tr(x)^q:

```
Tr(x)^q = x^q + x^q^2 + ... + x = Tr(x)
```

Tr(x)^q = Tr(x) means Tr(x) is in F_q (because x -> x^q is an automorhpism F_q^k -> F_q^k that preserves exactly F_q).

So we have a homomorphism from G2 to G1. The advantage of type 2 pairings is that any curve can be used and there is still a homomorphism from G2 to G1. On the other hand G2 does not have any special structure and it is impossible to sample randomly from G2 except by computing multiples of the generator, thus we cannot securely hash to G2. Also, membership testing can be a serious overhead.

The existence of homomorphism from G2 to G1 is sometimes require for security proofs. There are many schemes for which the security proof does not work if the system is implemented using type 3 pairings instead of type 2 pairings.

## Type 3

Let's have a pairing friendly curve E over F_q of embedding degree > 1. For G1 we take the subgroup of E(F_q) of order l. We can say G1 = ker(pi - [1]) ∩ E[l].

For G2 we take E[l] ∩ ker(pi - [q]). Note that [m] means x -> m*x in elliptic curve group. Thus, G2 are points on E[l] for which it holds:

```
(x^q, y^q) = q*(x,y)
```







[1] Galbraith, Steven D., Kenneth G. Paterson, and Nigel P. Smart. "Pairings for cryptographers." Discrete Applied Mathematics 156.16 (2008): 3113-3121.
