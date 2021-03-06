# Proof of discrete logarithm equality 

We have Schnorr group G_q and elements g1, h1, g2, h2. Prover knows a secret x such that:

```
h1 = g1^x and h2 = g2^x
```

Chaum-Pedersen protocol [1] proves the knowledge of dlog and the fact that it is the same: log_g1(h1) = log_g2(h2).

![dlog equality](https://raw.github.com/miha-stopar/crypto-notes/master/img/dlog_equality.png)

# Proof of discrete logarithm equality that produces a blinded transcript

![dlog equality_blinded_transcript](https://raw.github.com/miha-stopar/crypto-notes/master/img/dlog_equality_blinded_transcript.png)

We will see later how in a pseudonym system [2] a blinded transcript is used that proves the knowledge of dlog_g1(h1), dlog_g2(h2) and that dlog_g1(h1) = dlog_g2(h2). Blinded means that if a verifier runs the protocol several times, the prover will not be able to determine which conversation refers to which transcript.

Note that gamma is used because in [2] the verifier does not want to make public g2, h2 and thus he uses G2 = g2^gamma, H2 = h2^gamma.

Now, somebody who would like to check that the transcript is a valid proof for knowledge of log_g1(h1), log_G2(H2) and log_g1(h1) = log_G2(H2), he needs to check:

```
g1^(y+alpha) = A' * h1(c-beta)
G2^(y+alpha) = B' * H2(c-beta)
```

Also, hash needs to be checked (whether the second value in transcript is computed out of A' and B').

# Pseudonym systems [2]

Lysyanskaya et al. presented in 1999 a pseudonym system [2] which allow users to interact with multiple organizations anonymously, using pseudonyms. There were similar systems (for example Chaum [3]) proposed before, but they did not protect the system against dishonest users who collectively use their pseudonyms and credentials (they share the identity). Also, previous systems relied heavily on a trusted center.

Chaum [3] introduced a system in 1985 unlinkable pseudonyms (or nyms) where organizations cannot extract any knowledge on the user even if they combine their databases - a user can obtain a credential from one organization using one nym and demonstrate possession of the credential to another organization with a second nym, where the second organization cannot extract the first nym even if it colludes with a first organization. For example, Bob may get a credential asserting his good health from his doctor (who knows him by one nym), and show this to his insurance company (who knows him by another nym).

To motivate the user not to share his pseudonyms or credentials with other users, Lysyanskaya et al. [2] proposed to have a certificate authority (CA) to enable a user to prove to an organization that his pseudonym actually corresponds to a master public key of a real user - if the user shares a credential to that pseudonym, his master secret key can be computed.

A similar system was proposed by Caneti et al. [5], however the difference (to [2]) is that here organizations grant credentials only to users who reveal their identity to them (and then these credentials can be used anonymously to interact with other organizations).

## Registration with CA

Each user must first register with the CA, revealing his true identity and his master public key, and demonstrating possession of the corresponding master secret key. A user's pseudonym with the CA is his master public key.

A user thus obtains a credential from CA which is used in the registration with an organization (next step). This credential tells that the user has a valid master secret and public key (certifying that he has a valid identity).

## Registration with an organization (pseudonym generation)

User U's master secret key is x and master public key is g^x.

![nym generation](https://raw.github.com/miha-stopar/crypto-notes/master/img/nym_generation.png)

At the end user U and organisation O remembers U's nym: (a, b). 

Note that a is generated in the way that user does not know log_g(a). This is supposed to ensure that all user's nyms are tied to his secret x.

## Issuing a credential

User's nym with organization O: (a, b) where b = a^s and s is user's master secret.

Organization O's public key is (g, h1, h2) where h1=g^s1, h2=g^s2.

![nym credential_issue](https://raw.github.com/miha-stopar/crypto-notes/master/img/nym_credential_issue.png)

Note that the organization never sees the transcripts and it cannot link transcripts and users.

## Transferring a credential to another organization

User's nym with organization O1: (a1, b1) where b1 = a1^s.
User's credential from organization O is: (a'=a^gamma, b'=b^gamma, A'=A^gamma, B'=B^gamma, T1, T2).

![nym certificate_transfer](https://raw.github.com/miha-stopar/crypto-notes/master/img/nym_certificate_transfer.png)

User authenticates to the organization O1 by proving the knowledge of log_a1(b1). Using dlog equality proof user at the same time proves the knowledge of log_a'(b') - that the same master private key is used as for credential issue.

Organization then checks whether both transcripts are valid.

Note that the credential does not contain (a, b) in plain and thus the organization that verifies the credential cannot know the user's nym for the organization that issued a credential (this is why gamma is used in generation of the transcript).

# Proof of knowledge of a representation [6]

Let G be a finite cyclic group of prime order q and let g_1,...,g_k be generators of G (primality of q is actually not a necessary condition).

Tuple (x_1,...,x_k) is called a representation of y with respect to g_1,...,g_k when:

```
y = g_1^x_1 * ... * g_k^x_k
```

Proof is the generalization of Schnorr protocol. The prover chooses random r_1,...,r_k from Z_q, computes t = g_1^r_1 * ... * g_k^r_k, and sends t to the verifier. The verifier picks a random challenge c and sends it to the prover. The prover computes s_i = r_i - c * x_i mod q, and sends all s_i to the verifier. Verifier checks whether t = y^c * g_1^s_1 * ... * g_k^s_k.

# Non-transferable anonymous credentials [4]

While [2] discourage users from sharing their pseudonyms and credentials using PKI-assured non-transferability (sharing a credential implies also sharing a particular, valuable secret key from outside the system), Camenisch-Lysyanskaya [4] proposed a system which uses all-or-nothing non-transferability which does not require having some external valuable secret key. Here, sharing one pseudonym or credential implies sharing all of the user's other credentials and pseudonyms in the system.

Camenisch-Lysyanskaya credentials are somehow similar to the RSA signatures (note: this is simplified):

 * n = p * q; where p, q big primes
 * public key: (n, e), private key: (n, d); where e * d = 1 (mod Phi(n))
 * signature: m^d (mod n)
 * verification: (m^d)^e = m (mod n)

Camenisch-Lysyanskaya signature:

 * n = p * q; where p, q safe primes (safe primes to have a cyclic group of quadratic residues)
 * A = (Z / (R_1^m_1 * ... * R_l^m_l * S^v))^d (mod n); where Z, R_i, S are publicly known quadratic residues, v is random, e is random prime (as in RSA: e*d = 1 (mod Phi(n)))
 * signature (credential): (A, e, v)
 * verification: Z = A^e * (R_1^m_1 * ... * R_l^m_l * S^v) (mod n)

Example of Camenisch-Lysyanskaya credential:
 * credential (A, e, V) where A = (Z / (R_1^name * R_2^age * R_3^student_status * S^v))^d (mod n)
 * randomize the credential before it is used: new (A, e, V) = (A*S^r_A, e - 2^(l_e-1), v - e*r_a) where r_a is random

Reveal only student_status data for example to get a discount in a shop:
 * let the verifier know your student_status
 * the verifier computes Foo = Z / (R_3^student_status * A^e * S^v)
 * user proves via ZKP the knowledge of name and age such that Foo = R_1^name * R_2^age (mod n)
 * note that if user lied about student_status, he won't be able to produce a valid ZKP

# Signature scheme and anonymous credentials from bilinear maps

Camenisch-Lysyanskaya signature scheme [7] relies on LRSW assumption [2]: Let G be a cyclic group with generator g and of order |G|. Let g^x and g^y be given. Assume that an oracle can be called that answers a query s by a triple (a, a^(s*y), a^(x+s*x*y)), where a = g^z is a random group element of G. Let this oracle be called for s1,s2... Then, the problem is to generate (t, b, b^(t*y), b^(x+t*x*y), where t is not 0,s1,s2...

Actually, in [7] the triple given by oracle is (a, a^y, a^(x+s*x*y)), the quadruple to be generated is (t, b, b^y, b^(x+s*x*y).

The key idea in [7] is signature scheme for message m:

```
signature = (a, a^y, a^(x+m*x*y)) where a is random
```

The public key contains g^x, g^y. Verification:

```
e(a^y, g) = e(a, g^y)
e(g, a^(x+m*x*y)) = e(g^(x+m*x*y), a)
```

The right side of the second equations is computed:

```
e(g^(x+m*x*y), a) = e(g^x,a) * e*g^(m*x*y, a) = e(g^x, a) * e(g^x, a^y)^m
```

For anonymous credentials, a signature scheme needs to be randomized (the signature needs to be information-theoretically independent of the message m being signed). Thus the scheme above is modified in the following way - there is an additional value z in the secret key, g^z is part of the public key:

```
signature = (a, a^z, a^y, a^(z*y), a^(x+m*x*y) * a^(z*x*y*r)) where a is random and r is random
```

Finally, to enable the signatures on block of messages the scheme is modified:

```
signature = (a, a^z_1, ..., a^z_n, a^y, a^(z_1*y), ..., a^(z_n*y),  a^(x+m_0*x*y) * a^(z_1*x*y*m_1) * ... * a^(z_n*x*y*m_n)) where a is random and r is random
```







[1] D. Chaum and T. P. Pedersen, Wallet databases with observers, Advances in Cryptology — CRYPTO ’92 (E. F. Brickell, ed.), LNCS, vol. 740, Springer-Verlag, 1993, pp. 89– 105.

[2] A. Lysyanskaya, R. Rivest, A. Sahai, and S. Wolf. Pseudonym systems. In H. Heys and C. Adams, editors, Proceedings of SAC 1999, volume 1758 of LNCS, pages 184–99. SpringerVerlag, Aug. 1999.

[3] David Chaum. Security without identification: transaction systems to make Big Brother obsolete. Communications of the ACM, 28(10), 1985.

[4] Camenisch, J. and Lysyanskaya, A., “Efficient non-transferable anonymous multi-show credential system with optional anonymity revocation,” in EUROCRYPT 2001, 93-118. 2001.

[5] Ran Canetti, Moses Charikar, Ravi Kumar, Sridhar Rajagopalan, Amit Sahai, and Andrew Tomkins. Non-transferable anonymous credentials. Manuscript, 1998. Revision in submission, 1999.

[6] S. Brands. An efficient off-line electronic cash system based on the representation problem. Technical Report CS-R9323, CWI, Apr. 1993.

[7] Camenisch, Jan, and Anna Lysyanskaya. "Signature schemes and anonymous credentials from bilinear maps." Annual International Cryptology Conference. Springer, Berlin, Heidelberg, 2004.
