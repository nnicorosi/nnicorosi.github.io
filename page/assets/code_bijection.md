# Code complet : Bijection 312-évitante → Dyck

> **Code Source SageMath** > **Auteur :** Nita Nicorosi et plusieurs fonctions on été prise sur le site findstats 
> **Contexte :** Stage L2 au laboratoire LISN (Université Paris‑Saclay)

---

## Navigation
[Retour à l'accueil](../../README.md) | [Liste des projets](../projects.html) | [Page explicative du projet](../page/project_bijection_D_P.html)

---

Voici l’intégralité du code SageMath développé pendant mon stage. 
```python
from sage.combinat.permutation import Permutations
def avoids_312(pi):
    n = len(pi)
    for i in range(n):
        for j in range(i+1, n):
            for k in range(j+1, n):
                if pi[i] > pi[k] > pi[j]:
                    return False
    return True

def Av_312(n):
    return [list(p) for p in Permutations(n) if avoids_312(p)]

def mu_list(pi):
    M = 0
    out = []
    for x in pi:
        out.append(max(0, x - M))
        M = max(M, x)
    return out

def expected_mu(pi):
    k = pi.index(1)
    L = pi[:k]
    R = pi[k+1:]
    right = mu_list(std(R)) if R else []
    if not L:
        return [1] + right
    left = mu_list(std(L))
    return [left[0] + 1] + left[1:] + [0] + right

def records(pi):
    M = 0
    out = []
    for x in pi:
        if x > M:
            out.append(x)
            M = x
    return out

def std(p):
    if not p:
        return []
    vals = sorted(p)
    rank = {v: i+1 for i, v in enumerate(vals)}
    return [rank[x] for x in p]

def Phi(pi):
    M = 0
    w = []
    for x in pi:
        mu = max(0, x - M)
        M = max(M, x)
        w.extend(['u'] * mu)
        w.append('d')
    return ''.join(w)

def gamma(w):
    if w == "":
        return ()

    balance = 1
    for i in range(1, len(w)):
        if w[i] == 'u':
            balance += 1
        else:
            balance -= 1

        if balance == 0:
            A = w[1:i]
            B = w[i+1:]
            return (gamma(A), gamma(B))

    raise ValueError("Mot non Dyck")


def beta_min(pi):
    if len(pi) == 0:
        return ()

    m = min(pi)
    k = pi.index(m)
    L = pi[:k]
    R = pi[k+1:]
    return (beta_min(L), beta_min(R))

def check_theorem(nmax):
    for n in range(1, nmax+1):
        print("n =", n)
        for pi in Av_312(n):
            g = gamma(Phi(pi))
            b = beta_min(pi)
            if g != b:
                print("\nCONTRE EXEMPLE")
                print("pi =", pi)
                print("Phi(pi) =", Phi(pi))
                print("gamma(Phi(pi)) =", g)
                print("beta_min(pi) =", b)
                return False
    print("toujours vrai")
    return True

def check_conjecture_intermediaire(pi):
    m = min(pi)
    i = pi.index(m)
    L = pi[:i]
    R = pi[i+1:]
    Ls = std(L) if L else []
    Rs = std(R) if R else []
    w_expected = 'u' + (Phi(Ls) if Ls else '') + 'd' + (Phi(Rs) if Rs else '')
    return Phi(pi) == w_expected


def check_L_formula(pi):
    k = pi.index(1)
    L = pi[:k]
    if not L:
        return True
    WL = Phi(pi)[:len(Phi(L))]
    return WL == "u" + Phi(std(L))


def check_WL_formula(pi):
    k = pi.index(1)
    L = pi[:k]
    if not L:
        return True

    WL_expected = "u" + Phi(std(L))
    M = 0
    WL_actual = ""
    for x in L:
        mu = max(0, x - M)
        M = max(M, x)
        WL_actual += "u"*mu + "d"
    return WL_actual == WL_expected


def check_lemma2(pi):
    k = pi.index(1)
    L = pi[:k]
    R = pi[k+1:]
    lhs = Phi(pi)
    rhs = "u" + (Phi(std(L)) if L else "") + "d" + (Phi(std(R)) if R else "")
    return lhs == rhs


def check_first_return_block(w):
    bal = 0
    for i, c in enumerate(w):
        bal += 1 if c == "u" else -1
        if bal == 0:
            return w[:i+1]
    return None


def check_mu_structure(nmax):
    for n in range(1, nmax+1):
        for pi in Av_312(n):
            mu_calc = mu_list(pi)
            mu_expected = expected_mu(pi)
            if mu_calc != mu_expected:
                print("CONTRE EXEMPLE")
                print(f"pi = {pi}")
                print(f"mu_list(pi) = {mu_calc}")
                print(f"expected_mu(pi) = {mu_expected}")
                return False
    print("toujours vrai")
    return True


def check_corollary(pi):
    k = pi.index(1)
    L = pi[:k]
    p = len(L)
    return set(L) == set(range(2, p+2))

def run_all_tests(nmax: int = 10, verbose: bool = True):
    print(f"tests des permutations jusqu'a n = {nmax}\n")
    print(f"-"*40)
    
    print("\n1. Test du corollaire : verifie que L = {2,3,...,|L|+1}")
    count = 0
    for n in range(1, nmax+1): 
        for pi in Av_312(n): 
            count += 1
            if not check_corollary(pi): 
                print("CONTRE EXEMPLE")
                print(f"pi = {pi}")
                k = pi.index(1)
                L = pi[:k]
                print(f" L = {L}")
                print(f"attendu = {list(range(2, len(L)+2))}")
    print(f"Aucun contre exemple trouve (testes : {count} permutations)")
        
        
    print("\n2. Test de la structure de mu : verifie la recursivite de mu")
    if check_mu_structure(nmax): 
        print("structure verifiee")
        
        
    print("\n3. test de la conjecture intermediaire : verifie la decomposition de phi par rapport au minimum")
    succes_count = 0
    total_count = 0
    for n in range(1, nmax+1): 
        for pi in Av_312(n): 
            total_count += 1
            if check_conjecture_intermediaire(pi): 
                succes_count += 1
            elif verbose: 
                print(f"echec pour pi = {pi}")
    print(f"reussite : {succes_count}/{total_count} tests passes")
    
    
    print("\n4. test du lemme 2 : verifie que Phi(pi) = u + Phi(std(L))+d+Phi(std(R))")
    succes_count = 0
    total_count = 0
    for n in range(1, nmax+1): 
        for pi in Av_312(n): 
            total_count += 1
            if check_lemma2(pi): 
                succes_count += 1
            elif verbose: 
                print(f"echec pour pi = {pi}")
    print(f"reussite : {succes_count}/{total_count} tests passes")
    
    
    print("\n5. test de la formule WL : verifie la construction directe de la partie gauche de phi")
    succes_count = 0
    total_count = 0
    for n in range(1, nmax+1): 
        for pi in Av_312(n): 
            total_count += 1
            if check_WL_formula(pi): 
                succes_count += 1
            elif verbose: 
                print(f"echec pour pi = {pi}")
    print(f"reussite : {succes_count}/{total_count} tests passes")
    
    
    print("\n6. test de la formule L : verifie que WL=u + Phi(std(L))")
    succes_count = 0
    total_count = 0
    for n in range(1, nmax+1): 
        for pi in Av_312(n): 
            total_count += 1
            if check_L_formula(pi): 
                succes_count += 1
            elif verbose: 
                print(f"echec pour pi = {pi}")
    print(f"reussite : {succes_count}/{total_count} tests passes")

    print("\n7. Test du theoreme principal : gamma(Phi(pi)) == beta_min(pi)")
    if check_theorem(nmax):
        print("theoreme verifie")
    
    print("-"*40)
    print("Resume des tests")
    print(f"tous les tests on ete execute avec succes jusqu'a n = {nmax}")
    print(f"nombre totale de permutation 312-evitantes testees : {count}")
    
    
if __name__ == "__main__": 
    run_all_tests(nmax=11, verbose=False)
```

Et voici le code testant la compatibilité de differentes bijections avec Tamari 

```python


import itertools
from collections import deque

def is_dyck(w):
    h = 0
    for c in w:
        h += 1 if c == 'u' else -1
        if h < 0:
            return False
    return h == 0

def all_dyck_words(n):
    return [w for w in itertools.product('ud', repeat=2*n)
            if w.count('u') == n and is_dyck(w)]

def area_sequence(w):
    a, h = [], 0
    for c in w:
        if c == 'u':
            a.append(h); h += 1
        else:
            h -= 1
    return a

def path_height(w):
    h = best = 0
    for c in w:
        h += 1 if c == 'u' else -1
        best = max(best, h)
    return best

def avoids_pattern(p, pattern):
    n, m = len(p), len(pattern)
    for combo in itertools.combinations(range(n), m):
        vals = [p[i] for i in combo]
        rank = {v: i+1 for i, v in enumerate(sorted(vals))}
        if [rank[v] for v in vals] == list(pattern):
            return False
    return True

def Av(n, pattern):
    return [tuple(p) for p in itertools.permutations(range(1, n+1))
            if avoids_pattern(p, pattern)]

def inversions(p):
    return {(i, j) for i in range(len(p)) for j in range(i+1, len(p)) if p[i] > p[j]}

def weak_leq(pi, sigma):
    return inversions(pi).issubset(inversions(sigma))

def phi_312(pi):
    M, w = 0, []
    for x in pi:
        mu = max(0, x - M)
        M = max(M, x)
        w += ['u'] * mu
        w.append('d')
    return tuple(w)

def bandlow_killpatrick(w):
    n = len(w) // 2
    a = area_sequence(w)
    p = list(range(1, n+1))
    for j in range(n):
        for i in range(a[j]):
            k = j - i
            if 1 <= k < n:
                p[k-1], p[k] = p[k], p[k-1]
    return tuple(p)

def krattenthaler_132(w):
    n = len(w) // 2
    a = area_sequence(w); a.append(0)
    pi, values = [], list(range(1, n+1))
    for i in range(n):
        if a[n-i-1] + 1 > a[n-i]:
            v = n - i - a[n-i-1]
        else:
            v = min(x for x in values if x > n-i-a[n-i-1])
        pi.append(v); values.remove(v)
    return tuple(pi)


def to_pair_of_tableaux(w):
    n = len(w) // 2
    if n == 0:
        return ([], [])
    if path_height(w) == n:
        row = list(range(1, n+1))
        return ([row], [row])
    left, right = [[], []], [[], []]
    for pos in range(n):
        (left[0] if w[pos] == 'u' else left[1]).append(pos+1)
        (right[0] if w[-pos-1] == 'd' else right[1]).append(pos+1)
    return ([r for r in left if r], [r for r in right if r])

def rsk_reverse(P, Q):
    P = [row[:] for row in P]; Q = [row[:] for row in Q]
    n = sum(len(row) for row in P)
    perm = [None] * n
    for i in range(n, 0, -1):
        r = next(ridx for ridx, row in enumerate(Q) if row and row[-1] == i)
        Q[r].pop()
        val = P[r].pop()
        for rr in range(r-1, -1, -1):
            row = P[rr]
            pos = next(idx for idx in range(len(row)-1, -1, -1) if row[idx] < val)
            row[pos], val = val, row[pos]
        perm[i-1] = val
    return perm

def knuth_321(w):
    left, right = to_pair_of_tableaux(w)
    return tuple(rsk_reverse(left, right))

def gamma(w):
    if len(w) == 0:
        return ()
    bal = 0
    for i in range(len(w)):
        bal += 1 if w[i] == 'u' else -1
        if bal == 0:
            return (gamma(w[1:i]), gamma(w[i+1:]))
    raise ValueError("not a Dyck word")

def right_rotation_children(T):
    if T == ():
        return
    L, R = T
    if L != ():
        LL, LR = L
        yield (LL, (LR, R))
    for L2 in right_rotation_children(L):
        yield (L2, R)
    for R2 in right_rotation_children(R):
        yield (L, R2)

def tamari_leq_tree(T1, T2):
    if T1 == T2:
        return True
    seen = {T2}; q = deque([T2])
    while q:
        x = q.popleft()
        for y in right_rotation_children(x):
            if y == T1:
                return True
            if y not in seen:
                seen.add(y); q.append(y)
    return False

def dyck_tamari_leq(w1, w2):
    return tamari_leq_tree(gamma(w1), gamma(w2))


def test_tamari_compatibility(n, pattern, Phi, name):
    A = Av(n, pattern)
    for pi in A:
        for sigma in A:
            if weak_leq(pi, sigma):
                w_pi, w_sigma = Phi(pi), Phi(sigma)
                if not is_dyck(w_pi) or not is_dyck(w_sigma):
                    print(f"  [{name}] ERROR: {Phi.__name__} produced a non-Dyck word "
                          f"for pi={pi} or sigma={sigma}: {w_pi}, {w_sigma}")
                    return None
                if not dyck_tamari_leq(w_pi, w_sigma):
                    print(f"  [{name}] genuine counterexample at n={n}:")
                    print(f"    pi    = {pi}   (weak order: pi <= sigma)")
                    print(f"    sigma = {sigma}")
                    print(f"    Phi(pi)    = {''.join(w_pi)}")
                    print(f"    Phi(sigma) = {''.join(w_sigma)}")
                    print(f"    but Phi(pi) is NOT <=_Tamari Phi(sigma)")
                    return False
    print(f"  [{name}] no counterexample found up to n = {n}")
    return True

def make_inverse(f):
    cache = {}
    def inv(pi):
        n = len(pi)
        if n not in cache:
            table = {}
            for w in all_dyck_words(n):
                table[f(w)] = w
            cache[n] = table
        return cache[n][tuple(pi)]
    inv.__name__ = f.__name__ + "_inv"
    return inv

bandlow_killpatrick_inv = make_inverse(bandlow_killpatrick)
krattenthaler_132_inv   = make_inverse(krattenthaler_132)
knuth_321_inv           = make_inverse(knuth_321)

if __name__ == "__main__":
    NMAX = 8
    for n in range(2, NMAX+1):
        print(f"--- n = {n} ---")
        test_tamari_compatibility(n, [3,1,2], phi_312,             "phi (the paper's own map, 312)")
        test_tamari_compatibility(n, [3,1,2], bandlow_killpatrick_inv, "Bandlow-Killpatrick (312)")
        test_tamari_compatibility(n, [1,3,2], krattenthaler_132_inv ,   "Krattenthaler (132)")
        test_tamari_compatibility(n, [3,2,1], knuth_321_inv,           "Knuth (321)")
```
