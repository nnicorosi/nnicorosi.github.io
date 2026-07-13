# Bijection entre permutations 312‑évitantes et chemins de Dyck
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-MML-AM_CHTML"></script>

> **Projet de stage de L2 (Deux mois)** > **Laboratoire :** LISN (Université Paris‑Saclay)  
> **Encadrement :** Viviane Pons  
> **Thématique :** Description du treillis de Tamari sur les objets combinatoires.

Ce projet s’inscrit dans un stage de deux mois réalisé en L2 au laboratoire LISN (Université Paris‑Saclay), sous la direction de Viviane Pons. L’objectif général du stage était d’étudier le treillis de Tamari, une structure d’ordre fondamentale en combinatoire, et d’explorer ses liens avec différents objets de type Catalan.

---

## Contexte scientifique

Le treillis de Tamari apparaît dans de nombreux domaines : arbres binaires, chemins de Dyck, triangulations, permutations évitantes, structures algébriques, etc.

Dans ce stage, je me suis concentrée sur les permutations 312‑évitantes, qui sont en bijection avec les chemins de Dyck et les arbres binaires. Ces objets sont tous dénombrés par les nombres de Catalan, et leurs relations sont au cœur de la combinatoire moderne.

---

## Décomposition des permutations 312‑évitantes

Une permutation $$\pi$$ évite le motif $$312$$ si elle ne contient aucun triplet d'indices $$i < j < k$$ tel que :
$$\pi[k] < \pi[i] < \pi[j]$$

Ces permutations possèdent une décomposition naturelle par leur minimum :

$$\pi = L \cdot 1 \cdot R$$

où $$L$$ et $$R$$ sont eux‑mêmes des permutations 312‑évitantes. Cette décomposition est directement parallèle à la structure récursive des arbres binaires.

---

## La bijection construite

Le cœur de mon travail a été de définir une bijection explicite entre les permutations 312‑évitantes et les chemins de Dyck. Cette bijection, notée $\varphi$, est définie récursivement par :

$$\varphi(\pi) = u \cdot \varphi(\text{std}(L)) \cdot d \cdot \varphi(\text{std}(R))$$

Où :
* u représente une montée (up).
* d représente une descente (down).
* std désigne l'opérateur de standardisation.

Cette construction produit un chemin de Dyck de longueur $$2n$$, parfaitement compatible avec la structure du treillis de Tamari.

### Exemple concret

Pour la permutation $$\pi$$ = 3  1  2  4 :

* **Décomposition :** $$\pi = (3  2) \cdot 1 \cdot (4)$$
* **Arbre binaire associé :** $$Node(\beta(3  2), \beta(4))$$
* **Chemin de Dyck résultant $$(\varphi(\pi))$$ :** `u u d u d d`

---

## Compatibilité avec le treillis de Tamari

Une propriété essentielle que j’ai démontrée au cours de ce stage est la suivante :

$$\gamma(\varphi(\pi)) = \beta(\pi)$$

Autrement dit, la bijection transporte l’ordre faible droit sur les permutations 312‑évitantes vers l’ordre de Tamari sur les chemins de Dyck. C’est un résultat non trivial qui prouve que la bijection respecte la structure combinatoire profonde de ces objets.

---

## Apports du travail

* **Algorithmique :** Une bijection explicite, élégante et très simple à implémenter.
* **Théorique :** Une preuve directe et constructive de la compatibilité avec la structure de Tamari.
* **Combinatoire :** Une interprétation visuelle via les *records* de la permutation.
* **Expérimental :** Une base de code solide pour des expérimentations via SageMath (génération de tests, recherche de contre‑exemples et visualisations).

---

## Ressources & Documents

| Ressource | Description | Lien |
| :--- | :--- | :---: |
| **Article Complet** | Rapport détaillé incluant les démonstrations rigoureuses. | [Télécharger le PDF](assets/rewriting_bijection_312_avoing_permutation_dyck_path.pdf) |
| **Code Source** | Implémentation complète de la bijection et des outils de test. | [Voir le code Sage](assets/code_bijection.md) |

---
