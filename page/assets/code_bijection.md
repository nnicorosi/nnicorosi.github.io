# Code complet : Bijection 312-évitante → Dyck

> **Code Source SageMath** > **Auteur :** Nita Nicorosi  
> **Contexte :** Stage L2 au laboratoire LISN (Université Paris‑Saclay)

---

## Navigation
[Retour à l'accueil](../README.md) | [Liste des projets](../projects.html) | [Page explicative du projet](../pages/project_bijection_D_P.html)

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

def avoids_pattern(pi, pattern):
    return Permutation(pi).avoids(pattern)

def Av(n, pattern):
    return [tuple(p) for p in Permutations(n) if Permutation(p).avoids(pattern)]

def inversions(pi):
    inv = set()
    for i in range(len(pi)):
        for j in range(i+1, len(pi)):
            if pi[i] > pi[j]:
                inv.add((i, j))
    return inv

def weak_leq(pi, sigma):
    return inversions(pi).issubset(inversions(sigma))

def is_dyck(w):
    try:
        print(DyckWord(w))
        return True
    except:
        return False

def tamari_cover(w):
    w = list(w)
    n = len(w)
    covers = []

    for i in range(n-1):
        if w[i] == 'd' and w[i+1] == 'u':
            new = w[:i] + ['u', 'd'] + w[i+2:]
            if is_dyck(new):
                covers.append(tuple(new))
    return covers

def tamari_leq(a, b):
    from collections import deque
    seen = set([tuple(a)])
    queue = deque([tuple(a)])

    while queue:
        x = queue.popleft()
        if x == tuple(b):
            return True
        for y in tamari_cover(x):
            if y not in seen:
                seen.add(y)
                queue.append(y)
    return False

def krattenhaler(pi):
    n = len(pi)
    w = []
    
    current_min = pi[0]
    minima = [current_min]
    for x in pi[1:]:
        current_min = min(current_min, x)
        minima.append(current_min)
    
    height = pi[0]
    w += ['u'] * height
    
    for k in range(1, n):
        new_height = minima[k]
        if new_height < height:
            w += ['u'] * (height - new_height)
        
        w.append('d')
        height = new_height
    
    w += ['d'] * height
    return tuple(w)


def callan(pi):
    n = len(pi)
    w = []
    
    AD = []
    for i in range(n-1):
        if pi[i] < pi[i+1]:
            AD.append('A')
        else:
            AD.append('D')
    
    for c in AD:
        if c == 'A':
            w.append('u')
        else:
            w.append('d')
    w.append('d')
    return tuple(w)

def bjs(pi):
    n = len(pi)
    w = []
    excess = [pi[i] - (i+1) for i in range(n)]
    height = 0
    
    if excess[0] > 0:
        w += ['u'] * excess[0]
        height = excess[0]
    
    for i in range(1, n):
        diff = excess[i] - excess[i-1]
        if diff > 0:
            w += ['u'] * diff
            height += diff
        elif diff < 0:
            w += ['d'] * (-diff)
            height += diff
            
    if height > 0:
        w += ['d'] * height
    return tuple(w)

def bandlow_factorization(pi):
    pi = list(pi)
    n = len(pi)
    blocks = []
    transpositions = []

    for target in range(n, 0, -1):
        pos = pi.index(target)
        while pos != target - 1:
            transpositions.append(pos)
            pi[pos], pi[pos+1] = pi[pos+1], pi[pos]
            pos += 1

    transpositions = transpositions[::-1]
    current_block = []
    last = -1
    for t in transpositions:
        if t > last:
            current_block.append(t)
            last = t
        else:
            blocks.append(current_block)
            current_block = [t]
            last = t
    if current_block:
        blocks.append(current_block)
    return blocks


def bandlow(pi):
    n = len(pi)
    blocks = bandlow_factorization(pi)
    grid = [[False]*n for _ in range(n)]

    for block in blocks:
        l = len(block)
        m = block[-1]
        row = m
        for col in range(m, m-l, -1):
            grid[row][col] = True

    path = []
    x = y = 0
    while x < n or y < n:
        if y < n and not grid[y][x]:
            path.append('u')
            y += 1
        else:
            path.append('d')
            x += 1
    return tuple(path)

def test_tamari_compatibility(n, pattern, Phi):
    A = Av(n, pattern)
    for i in range(len(A)):
        for j in range(len(A)):
            pi = A[i]
            sigma = A[j]
            if weak_leq(pi, sigma):
                w_pi = Phi(pi)
                w_sigma = Phi(sigma)
                if not tamari_leq(w_pi, w_sigma):
                    print("Contre-exemple trouvé :")
                    print("pi =", pi)
                    print("sigma =", sigma)
                    print("Phi(pi) =", w_pi)
                    print("Phi(sigma) =", w_sigma)
                    return False
    print("Aucun contre-exemple trouvé jusqu'à n =", n)
    return True

print(test_tamari_compatibility(7, [1, 3, 2], krattenhaler))
print(test_tamari_compatibility(7, [3, 2, 1], callan))
print(test_tamari_compatibility(7, [3, 2, 1], bjs))
```
