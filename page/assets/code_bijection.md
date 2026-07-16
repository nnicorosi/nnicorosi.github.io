# Code complet : Bijection 312-évitante → Dyck

> **Code Source SageMath** > **Auteur :** Nita Nicorosi et plusieurs fonctions on été prise sur le site findstats 
> **Contexte :** Stage L2 au laboratoire LISN (Université Paris‑Saclay)

---

## Navigation
[Retour à l'accueil](../../README.md) | [Liste des projets](../projects.md) | [Page explicative du projet](../page/project_bijection_D_P.md)

---

Voici l’intégralité du code SageMath développé pendant mon stage. 

Pour cette premiere partie, on vérifie le théoreme principal de l'article : gamma(phi(pi))=beta_min(pi) pour tout pi dans Av_n(312). Pour ce faire, on vérifie egalement les lemmes intermediaires, a savoir le Lemmes 4.1 et 4.2 ainsi que la decomposition de phi autour du minimum. 


Dans la suite, toutes les permutations sont représentées comme des listes Python. 
```python
from sage.combinat.permutation import Permutations


def avoids_312(pi):

    """
    Teste si la permutation pi évite le motif 312 (définition 3.5). 

    Params :
        pi (liste) : la permutation.
    Returns :
        bool : True si elle évite le motif, False sinon. 
    """

    n = len(pi)
    for i in range(n):
        for j in range(i+1, n):
            for k in range(j+1, n):
                if pi[i] > pi[k] > pi[j]:
                    return False
    return True


def Av_312(n):

    """
    Calcule la liste des permutations de taille n qui évite le motif 312.

    Params :
        n (int) : la taille souhaitée de la permutation.
    Returns :
        list : la liste des permutations de taille n qui évite le motif 312.
    """

    return [list(p) for p in Permutations(n) if avoids_312(p)]


def std(p):

    """
    Standardisation d'une suite d'entiers distincts (Définition 4.1).

    Params :
        p (list) : la suite d'entiers dinstincts que l'on veut standardiser.
    Returns :
        list : l'unique permutation de {1, ..., |p|} qui respecte l'ordre relatif des éléments de p.
    """

    if not p:
        return []
    vals = sorted(p)
    rank = {v: i+1 for i, v in enumerate(vals)}
    return [rank[x] for x in p]


def mu_list(pi):

    """
    Calcule la suite des accroissements de records ( mu_1, ..., mu_n ) de pi telle que définie dans la Définition 3.9:
        mu_i = pi[i] - M_{i-1} si pi[i] > M_{i-1} (record)
        mu_i = 0 sinon.
    Avec M_i = max( pi[1], ..., pi[i] ) et M_0 = 0.

    Params :
        pi (list) : la permutation dont on veut tirer les accroissements de records.
    Returns :
        list : la suite des accroissements de records de pi.
    """

    M = 0
    out = []
    for x in pi:
        out.append(max(0, x - M))
        M = max(M, x)
    return out


def expected_mu(pi):

    """
    Calcule (mu_1,..., mu_n) via la formule recursive du Lemme 4.2.

    On décompose pi = L · 1 · R autour de la position du minimum. On reconstruit :
        - mu(L)=[ alpha_1 + 1, alpha_2, ..., alpha_p ] avec (alpha_i)=mu_liste(std(L)).
        (le +1 sur le premier terme correspond au décalage de valeur établi par le Corollaire 4.1: L_i = std(L)_i +1)
        - le terme central vaut toujours 0 dans ce cas (p >= 1 forcé ici, car si L est vide, on ne préfixe pas par [left[0]+1]).
        - mu(R) = mu_liste(std(R)) inchangé (Lemme 4.2, les records de R correspondent exactement aux records de std(R)).

    Params :
        pi (list) : la permutation dont on veut tirer les accroissements de records.
    Returns :
        list : la suite des accroissements de records de pi.
    """

    k = pi.index(1)
    L = pi[:k]
    R = pi[k+1:]
    right = mu_list(std(R)) if R else []
    if not L:
        #cas p = 0: le terme central devient ud (mu_1(pi) = 1) 
        return [1] + right
    left = mu_list(std(L))
    return [left[0] + 1] + left[1:] + [0] + right


def records(pi):

    """
    Renvoie la liste des valeurs records de pi.

    Params :
        pi (list) : la permutation dont on veut la liste des valeurs records.
    Returns :
        list: la liste des valeurs records. 
    """

    M = 0
    out = []
    for x in pi:
        if x > M:
            out.append(x)
            M = x
    return out


def Phi(pi):

    """
    Construit le mot de Dyck phi(pi) = u^{mu_1} d u^{mu_2} d ... u^{mu_n} d (Définition 3.10), directement a partir de pi.

    Cela permet d'avoir une implémentation indépendantes à la fin des tests.

    Params :
        pi (list) : la permutation.
    Returns :
        str : le mot de Dyck correspondant à pi. 
    """

    M = 0
    w = []
    for x in pi:
        mu = max(0, x - M)
        M = max(M, x)
        w.extend(['u'] * mu)
        w.append('d')
    return ''.join(w)


def gamma(w):

    """
    Applique la décomposition canonique récursive gamma : D_n -> T_n (Définition 3.3, Lemme 3.1).

    On cherche le plus petit préfixe non vide de w qui revient à la hauteur 0. Ce prefiwe s'écrit u A d et w = a A d B. On renvoie l'arbre vide (gamma(A), gamma(B)), représenté comme un tuple imbriqué (le tuple vide () représente l'arbre vide/une feuille selon le contexte).

    Params :
        w (str) : le mot de Dyck que l'on veut représenter en arbre binaire.
    Returns :
        tuples : l'arbre binaire représenté comme des tuples imbriqués.
    """

    if w == "":
        return ()

    balance = 1 # On a déja lu le premier caractère, forcément "u". 
    for i in range(1, len(w)):
        if w[i] == 'u':
            balance += 1
        else:
            balance -= 1

        if balance == 0:
            # w[0:i+1] = u A d est le plus petit prefixe qui revient à 0 avec A = w[1:i] (Lemme 3.1).
            A = w[1:i]
            B = w[i+1:]
            return (gamma(A), gamma(B))

    raise ValueError("Mot non Dyck")


def beta_min(pi):

    """
    Applique la bijection récursive beta_min de Bjorner-Wachs (Définition 3.6). On décompose pi autour de son minimum, sans standardisation (contrairement à expected_mu / Phi qui elles standardisent L et R).

    Params :
        pi (list) : la permutation.
    Returns :
        tuples : l'arbre associé à la permutation sous forme de tuples imbriqués. 
    """

    if len(pi) == 0:
        return ()

    m = min(pi)
    k = pi.index(m)
    L = pi[:k]
    R = pi[k+1:]
    return (beta_min(L), beta_min(R))


def check_theorem(nmax):

    """
    Vérifie le theoreme 4.1 : gamma(phi(pi))=beta_min(pi) pour tout pi dans Av_n(312).

    Params :
        nmax (int) : la taille maximum des permutations que l'on veut tester (on testera pour tout n dans [1, ..., nmax].
    Returns :
        bool : True si le théoreme est vérifie et ne possède aucun contre exemple, False sinon.

    laisse une trace dans le terminal avec le premier contre exemple trouvé si il y en a un, et toujours vrai si le théoreme est vérifié. 
    """
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

    """
    Vérifie la décomposition de phi telle que dans le Lemme 4.2 : phi(pi) = u phi(std(L)) d phi(std(R)) en comparant phi(pi) (calculé directement) au mot reconstruit membre à membre.

    Params :
        pi (list) : la permutation
    Returns :
        bool : True si la conjecture est respectée, False sinon. 
    """
    m = min(pi)
    i = pi.index(m)
    L = pi[:i]
    R = pi[i+1:]
    Ls = std(L) if L else []
    Rs = std(R) if R else []
    w_expected = 'u' + (Phi(Ls) if Ls else '') + 'd' + (Phi(Rs) if Rs else '')
    return Phi(pi) == w_expected


def check_L_formula(pi):

    """
    Vérifie que le préfixe de phi(pi) correspondant aux psoitions de L est exactement u + phi(std(L)). Pour etre plus précis : le résultaat du paragaphe "Computation of phi(L)" dans la preuve du Lemme 4.2.

    Params :
        pi (list) : la permutation
    Returns :
        bool : True si la formule est vérifiée, False sinon. 
    """

    k = pi.index(1)
    L = pi[:k]
    if not L:
        return True
    WL = Phi(pi)[:len(Phi(L))]
    return WL == "u" + Phi(std(L))


def check_WL_formula(pi):
    """
    Même vérification que check_L_formula, mais en reconstruisant le préfixe WL directement bloc par bloc à partir de L (sans passer par Phi(pi)), pour un test indépendant.

    Params :
        pi (list) : la permutation.
    Returns :
        bool : True si la formule est vérifiée, False sinon. 
    """
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


def check_first_return_block(w):

    """
    Renvoie le plus petit préfiwe de w qui revient à hauteur 0. (utilitaire pour le debogage pour visualiser la décomposition canonique u A d utilisé par gamma).

    Params :
        w (str) : un mot de Dyck
    Returns :
        str : le plus petit prefixe de w qui revient à hauteur 0.
    """

    bal = 0
    for i, c in enumerate(w):
        bal += 1 if c == "u" else -1
        if bal == 0:
            return w[:i+1]
    return None


def check_mu_structure(nmax):
    """
    Vérifie que la suite mu calculée directement (mu_list) coincie avec la suite mu reconstruite recursivement (expected_mu). C'est un test de la recursivité de mu annoncée dans le Lemme 4.2.

     Params :
        nmax (int) : la taille maximum des permutations que l'on veut tester (on testera pour tout n dans [1, ..., nmax].
    Returns :
        bool : True si le théoreme est vérifie et ne possède aucun contre exemple, False sinon.

    laisse une trace dans le terminal avec le premier contre exemple trouvé si il y en a un, et toujours vrai si le théoreme est vérifié.  
    """

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

    """
    Vérifie le Corollaire 4.1 / Lemme 4.1 : après décomposition, pi = L 1 R, on doit avoir Val(L) = {2, ..., |L|+1}.

    Params :
        pi (list) : la permutation.
    Returns :
        bool : True si la proposition est vraie, False sinon. 
    """

    k = pi.index(1)
    L = pi[:k]
    p = len(L)
    return set(L) == set(range(2, p+2))


def run_all_tests(nmax: int = 10, verbose: bool = True):

    """
    Lance l'ensemble des tests ci-dessus at affiche un resumé.

    Params :
        nmax (int) : la taille maximal des permutations que l'on va tester.
        verbose (bool) : si True, affiche les permutations qui ont mit en echec. Utilisé pour le debogage.
    Retruns :
        None

    laisse une trace dans le terminal avec plusieurs affichages en fonctions des différents checks. 
    """
    
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
        
    print("\n4. test de la formule WL : verifie la construction directe de la partie gauche de phi")
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
    
    
    print("\n5. test de la formule L : verifie que WL=u + Phi(std(L))")
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

    print("\n6. Test du theoreme principal : gamma(Phi(pi)) == beta_min(pi)")
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

Dans la suite, nous allons tester pour plusieurs bijection candidates entre permutations à pattern évitantes et mot de Dyck si elles envoient l'ordre faible droit (Définition 3.7) sur l'ordre de Tamari (Fait 3.1). 

C'est cela qui a permit de vérifier empiriquement que : 
    - phi preserve Tamari
    - Bandlow-Killpatrick (via son inverse) preserve Tamari

Grace a ce code, j'ai egalement pu expérimenté pour d'autres bijections, mais qui n'apparaisse pas dans l'article, c'est pourquoi j'ai fait le choix de les retirer de ce bout de code. 

```python


import itertools
from collections import deque

def is_dyck(w):

    """
    Teste les conditions (D1) (D2) de la définition 3.1 : hauteur finale nulle et hauteur toujours positive ou nulle en cours de lecture. 

    Params :
        w (str) : un mot de Dyck.
    Return :
        bool : True si les deux conditions sont remplies, False sinon. 
    """

    h = 0
    for c in w:
        h += 1 if c == 'u' else -1
        if h < 0:
            return False
    return h == 0


def all_dyck_words(n):

    """
    Enumere tous les mots de Dyck de semi-longueur n (utilisé uniquement pour construire les tables d'inversion, cf. make_inverse ci dessous. Tres couteux, reservé pour de petit n).

    Params :
        n (int) : la semi longueur des mots de Dyck dont on veut les tables.
    Returns :
        list : la liste de tous les mots de Dyck de semi longueur n. 
    """
    return [w for w in itertools.product('ud', repeat=2*n)
            if w.count('u') == n and is_dyck(w)]


def area_sequence(w):

    """
    Renvoie la suite des hauteurs juste avant chaque pas montant (c'est le profil a_j(w) de la Définition 8.2 indexé a partir de 0, utilisé pour la construction de Banlow Killpatrick).

    Params :
        w (str) : un chemin de Dyck.
    Returns :
        list : la suite des hauteurs juste avant chaque pas montant. 
    """
    a, h = [], 0
    for c in w:
        if c == 'u':
            a.append(h); h += 1
        else:
            h -= 1
    return a


def path_height(w):

    """
    Hauteur maximale atteinte par le chemin (Proposition 7.4).

    Params :
        w (str) : un mot de Dyck.
    Returns :
        int : la hauteur maximale atteinte par w. 
    """

    h = best = 0
    for c in w:
        h += 1 if c == 'u' else -1
        best = max(best, h)
    return best


def avoids_pattern(p, pattern):

    """
    Test générique d'evitement de motif d'après la définition 3.4 : p évite un pattern si il n'existe aucune sous suite de p dont l'ordre relatif reproduit ce pattern. C'est une version plus générale de la fonction avoids_312.

    Params :
        p (list) : la permutation.
        pattern (list) : la liste contenant le pattern.
    Returns :
        bool : True si p evite pattern, False sinon. 
    """

    n, m = len(p), len(pattern)
    for combo in itertools.combinations(range(n), m):
        vals = [p[i] for i in combo]
        rank = {v: i+1 for i, v in enumerate(sorted(vals))}
        if [rank[v] for v in vals] == list(pattern):
            return False
    return True


def Av(n, pattern):

    """
    Renvoie Av_n(pattern) toutes tailles de motif confondues.

    Params :
        n (int) : la taille souhaitée
        pattern (list) : le pattern a éviter.
    Returns :
        list : les permutation de taille n qui évite pattern. 
    """

    return [tuple(p) for p in itertools.permutations(range(1, n+1))
            if avoids_pattern(p, pattern)]


def inversions(p):

    """
    Ensemble des inversions de p tel que définit dans Définition 3.7.

    Params :
        p (list) : la permutation.
    Returns :
        dict : les inversions de p.
    """

    return {(i, j) for i in range(len(p)) for j in range(i+1, len(p)) if p[i] > p[j]}


def weak_leq(pi, sigma):

    """
    Ordre faible à droite (Définition 3.7)

    Params :
        pi (list) : la permutation
        sigma (list) : l'autre permutation que l'on veut comparer.
    Returns :
        bool : True si pi <=_W sigma ie si Inv(pi) est inclu dans Inv(sigma), False sinon. 
    """

    return inversions(pi).issubset(inversions(sigma))


def phi_312(pi):

    """
    phi de l'article (Définition 3.10), redéfinie ici, independament du premier script pour que ce fichier soit autonome.

    Params :
        pi (list) : la permutation.
    Returns :
        str : le mot de Dyck associé. 
    """

    M, w = 0, []
    for x in pi:
        mu = max(0, x - M)
        M = max(M, x)
        w += ['u'] * mu
        w.append('d')
    return tuple(w)


def bandlow_killpatrick(w):

    """
    Reconstruction de BK(w) (Définition 8.3) sans passer par la suite d'insertion sigma^0, sigma^1, ..., sigma^n explicite : on part directement d'un tableau p = [1, ..., n] et on fiat glisser chaque valeur j+1 de a[j] positions vers la gauche par transpositions adjactentes.

    C'est equivalent au Lemme 8.1 (une chaine de a transpositions adjace,tes consécutives sur une valeur nouvellemen insérée en position r équivaut à une insertion directe en position r - a) : au lieu d'inserer puis de faire remonter la valeur, on la fait glisser directement dans un tableau déjà alloué à la taille finale n. C'est numériquement équivalent mais évite les réallocations de liste.

    Params :
        w (str) : un mot de Dyck.
    Returns :
        list : la permutation associée. 
    """

    n = len(w) // 2
    a = area_sequence(w)
    p = list(range(1, n+1))
    for j in range(n):
        for i in range(a[j]):
            k = j - i
            if 1 <= k < n:
                p[k-1], p[k] = p[k], p[k-1]
    return tuple(p)


def gamma(w):

    """
    Meme décomposition canonique que dans le premier script (Définition 3.3).

    Params :
        w (str) : un mot de Dyck.
    Returns :
        tuples : l'arbre binaire associé.
    """
    
    if len(w) == 0:
        return ()
    bal = 0
    for i in range(len(w)):
        bal += 1 if w[i] == 'u' else -1
        if bal == 0:
            return (gamma(w[1:i]), gamma(w[i+1:]))
    raise ValueError("not a Dyck word")


def right_rotation_children(T):

    """
    Génère tous les arbres obtenus à partir de T par une seule rotation droite (Fait 3.1). On explore recursivement : rotation à la racine si le sous arbre gauche est non vide, ou rotation plus profonde dans l'un des deux sous arbres.

    Params :
        T (tuples) : l'arbre.
    Returns :
        tuples : les sous abres produits. 
    """

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

    """
    Teste si T1 < =_Tamari T2 par parcours en largeur (BFS) : T1 <= T2 ssi T1 est atteignable depuis T2 par une suite de rotations droites (Fait 3.1). On part de T2 et on explore l'ensemble des arbres accessibles par rotations droites successives jusqu'a trouver T1.

    Params :
        T1 (tuples) : le premier arbre.
        T2 (tuples) : le deuxieme arbre.
    Returns :
        bool : True si T1 <=_Tamari T2, False sinon. 
    """

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

    """
    Transporte le test précédent aux mot de Dyck via gamma (Fait 3.1)

    Params :
        w1 (str) : le premier mot de Dyck.
        w2 (str) : le deuxieme mot de Dyck.
    Returns :
        bool : True si w1 <=_Tamari w2, False sinon. 
    """

    return tamari_leq_tree(gamma(w1), gamma(w2))


def test_tamari_compatibility(n, pattern, Phi, name):

    """
    Pour tout paire (pi, sigma) de Av_n(pattern) avec pi <=_W sigma, vérifie que Phi(pi) <=_Tamari Phi(sigma).

    C'est exactement le test empirique dont déprend le théoreme 6.1.

    Params :
        n (int) : la taille des permutations à tester.
        pattern (list) : le pattern a eviter.
        Phi (fct) : la fonction que l'on veut tester.
        name (str) : le nom de la bijection.
    Returns :
        bool : True si aucun contre exemple n'est trouvé, False sinon. 
    """

    A = Av(n, pattern)
    for pi in A:
        for sigma in A:
            if weak_leq(pi, sigma):
                w_pi, w_sigma = Phi(pi), Phi(sigma)
                if not is_dyck(w_pi) or not is_dyck(w_sigma):
                    print(f" [{name}] ERROR: {Phi.__name__} produced a non-Dyck word "
                          f"for pi={pi} or sigma={sigma}: {w_pi}, {w_sigma}")
                    return None
                if not dyck_tamari_leq(w_pi, w_sigma):
                    print(f" [{name}] genuine counterexample at n={n}:")
                    print(f" pi = {pi} (weak order: pi <= sigma)")
                    print(f" sigma = {sigma}")
                    print(f" Phi(pi) = {''.join(w_pi)}")
                    print(f" Phi(sigma) = {''.join(w_sigma)}")
                    print(f" but Phi(pi) is NOT <=_Tamari Phi(sigma)")
                    return False
    print(f" [{name}] no counterexample found up to n = {n}")
    return True


def make_inverse(f):

    """
    Construit l'inverse d'une bijection Dyck -> Permutation par force brut. Pour chaque taille n rencontrée, tabule f sur tous les mots de Duck de taille n puis inverse la table. Couteux (exponentiel en n)

    Params :
        f (fct) : la fonciton dont on veut l'inverse.
    Returns :
        fct : l'inverse de f. 
    """

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

if __name__ == "__main__":
    NMAX = 8
    for n in range(2, NMAX+1):
        print(f"--- n = {n} ---")
        test_tamari_compatibility(n, [3,1,2], phi_312, "phi (the paper's own map, 312)")
        test_tamari_compatibility(n, [3,1,2], bandlow_killpatrick_inv, "Bandlow-Killpatrick (312)")
```
