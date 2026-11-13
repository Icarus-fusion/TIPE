# TIPE
 
Ce projet est une simulation 2D en Python d'un plasma de fusion nucléaire Deutérium–Tritium (D-T) confiné dans un réacteur de type tokamak. Le projet a été réalisé dans le cadre d'un TIPE.
 
Le programme simule un ensemble de particules (D et T) qui se déplacent dans un tore « déplié » en rectangle, interagissent par la force électrostatique et peuvent fusionner lorsqu'elles se rapprochent suffisamment, en fonction de leur énergie cinétique et de la section efficace de la réaction D-T.
 
---
 
## Sommaire
 
- [Principe](#principe)
- [Modèle physique](#modèle-physique)
- [Paramètres](#paramètres)
- [Installation et utilisation](#installation-et-utilisation)
- [Fichiers de sortie](#fichiers-de-sortie)
- [Structure du code](#structure-du-code)
- [Traitement des données](#traitement-des-données)
---
 
## Principe
 
1. Un nombre `N` de particules est créé, avec une proportion `rdt` de Deutérium et le reste en Tritium.
2. Chaque particule reçoit une position aléatoire, une direction aléatoire et une **vitesse tirée selon la distribution de Maxwell-Boltzmann** à la température du plasma.
3. À chaque itération :
   - on détecte les **fusions** (distance entre un D et un T inférieure à la somme de leurs rayons de section efficace) ;
   - on calcule la **force de Coulomb** exercée par toutes les autres particules, puis on met à jour vitesse et position.
4. À chaque fusion, un compteur est incrémenté, l'instant est enregistré, et les deux particules sont **réinjectées** à des positions fixes avec de nouvelles vitesses tirées dans la distribution de Maxwell (le nombre de particules reste donc constant).
## Modèle physique
 
### Géométrie du tore
 
Le volume simulé est proportionnel au nombre de particules, de façon à conserver la densité d'un réacteur de type ITER (840 m³ pour 4,2·10²² particules, soit ≈ 5·10¹⁹ m⁻³) :
 
- `V = 840 · N / 4,2e22`
- Rapport d'aspect `R / r = 3,1`, d'où le petit rayon `r = (V / (2π² · 3,1))^(1/3)` et le grand rayon `R = 3,1 · r`
- Le tore est représenté par un rectangle de **1920 × 620 pixels** aux bords périodiques. L'échelle est `m_p = 2πR / 1240` mètres par pixel.
### Distribution des vitesses
 
La vitesse est tirée avec `random.choices` selon la densité de probabilité de Maxwell-Boltzmann :
 
$$ f(v) = \sqrt{\frac{2}{\pi}}\left(\frac{m}{k_BT}\right)^\frac{3}{2}v^2\exp\left(\frac{-mv^2}{2k_BT}\right) $$
 
L'axe des vitesses est discrétisé en `N_sub` points entre 0 et `V_MAX`.
 
### Section efficace
 
La section efficace D-T est approchée par une **interpolation log-linéaire par morceaux** de la courbe expérimentale (énergie en keV, plage de 2,3 keV à 5 MeV, nulle en dehors). Elle est convertie en rayon $\displaystyle\sqrt{\frac{\sigma}{\pi}}$, puis en pixels.
 
### Interaction électrostatique
 
Chaque particule subit la force de Coulomb répulsive de toutes les autres :
 
$$ F = \frac{e^2}{4\pi\varepsilon_0d^2} $$
 
avec prise en compte de la périodicité du domaine pour le calcul des distances.
 
### Intégration
 
Les accélérations, vitesses et positions sont mises à jour par la **méthode des trapèzes** (fonction `integration`).
 
## Paramètres
 
| Paramètre | Valeur par défaut | Description |
|---|---|---|
| `N` | `10000` | Nombre de particules simulées |
| `rdt` | `0.5` | Proportion de particules de Deutérium |
| `T` | `1.5e8` K | Température du plasma |
| `Taille` | `2e-15` m | Taille d'un atome d'hydrogène |
| `V_MAX` | `1.5e7` m/s | Vitesse maximale considérée (50 % de c) |
| `N_sub` | `10000` | Nombre de subdivisions pour la courbe des vitesses |
| `Largueur` × `Hauteur` | `1920 × 620` px | Dimensions du domaine simulé |
 
Constantes physiques utilisées : `k_b`, `epsilon_0`, `e`, `M_d`, `M_t`.
 
## Installation et utilisation
 
### Prérequis
 
- Python 3.8 ou supérieur
- Aucune dépendance externe pour la simulation (uniquement la bibliothèque standard : `random`, `time`, `math`)
- Le traitement des données nécessite en revanche des bibliothèques tierces (voir [Traitement des données](#traitement-des-données))
### Configuration des chemins
 
Les fichiers de sortie sont écrits dans des chemins **codés en dur**. Avant de lancer le programme, modifiez-les dans le code (`Info.txt`, `Fusion.txt`, `Instant.txt`) pour qu'ils pointent vers un dossier existant sur votre machine.
 
### Lancement
 
Le fichier ne contient que des définitions : il faut ajouter un point d'entrée à la fin de `VO.py` pour lancer la simulation.
 
```python
if __name__ == "__main__":
    Reacteur(iteration=100)   # nombre de pas de simulation
```
 
Puis :
 
```bash
python VO.py
```
 
À chaque fusion, la console affiche le numéro de la fusion et le temps écoulé depuis le début.
 
## Fichiers de sortie
 
| Fichier | Contenu |
|---|---|
| `Info.txt` | Récapitulatif des dimensions (volume, rayons, échelle) et des paramètres de la simulation (en mode ajout) |
| `Fusion.txt` | Numéro de chaque fusion, une par ligne |
| `Instant.txt` | Instant (en secondes depuis le début) de chaque fusion, une par ligne |
 
Ces deux derniers fichiers permettent de tracer le nombre cumulé de fusions en fonction du temps.
 
## Structure du code
 
| Élément | Rôle |
|---|---|
| `section_efficace(E)` | Rayon de la section efficace D-T (m) en fonction de l'énergie (keV) |
| `maxwell(v, particule)` | Densité de probabilité de Maxwell-Boltzmann pour D ou T |
| `vitesse(particule)` | Tirage aléatoire d'une vitesse selon cette distribution |
| `integration(X, Y)` | Intégration par la méthode des trapèzes (dernier intervalle) |
| `class Particule` | État d'une particule (position, vitesse, type, masse, section) ; méthodes `distance`, `force_electrostatique`, `section_efficace`, `move` |
| `class Reacteur` | Création des particules, détection des fusions (`fusion`) et boucle de simulation (`animation`) |
 
## Traitement des données
 
Le script `Traitement_des_donnees.py` exploite les fichiers produits par la simulation (`Instant.txt` et `Fusion.txt`) pour estimer le **taux de fusion** du plasma, puis en déduire une **puissance de fusion** extrapolée à l'échelle du réacteur.
 
### Principe
 
1. Les fichiers `Instant.txt` (instant de chaque fusion) et `Fusion.txt` (nombre cumulé de fusions) sont lus avec `load_list`.
2. Une **régression linéaire** (`SGDRegressor` de scikit-learn) est ajustée sur la courbe *nombre de fusions = f(temps)*. La pente `a` correspond au taux de fusion (nombre de fusions par seconde) et `b` à l'ordonnée à l'origine.
3. La fonction `puissance(a, V)` convertit ce taux en puissance :
$$ P = \frac{a \cdot E_{fusion}}{V} \times 840 $$
 
   où `E_fusion` est l'énergie libérée par fusion (en joules, valeur `2.256e-13` codée en dur dans le script), `V` le volume simulé (en m³) et `840` le volume du plasma d'ITER (en m³). On obtient ainsi la puissance (en W) qu'aurait un plasma de même densité occupant tout le volume du réacteur.
4. Un graphique compare les données simulées à la droite de régression (en rouge).
 
### Prérequis
 
```bash
pip install numpy matplotlib scikit-learn
```
 
### Utilisation
 
1. **Lancer d'abord la simulation** (`VO.py`) afin de générer `Instant.txt` et `Fusion.txt`.
2. Dans `Traitement_des_donnees.py`, adapter les chemins des fichiers `Instant.txt` et `Fusion.txt`. 
3. Adapter le volume passé à `puissance` : il doit correspondre au volume simulé `V` indiqué dans `Info.txt` (`2e-16` m³ pour `N = 10000`).
```python
print(puissance(a, 2e-16))
```
 
4. Lancer le script :
```bash
python Traitement_des_donnees.py
```
 
### Résultats
 
| Sortie | Signification |
|---|---|
| `a, b` (console) | Pente (fusions/s) et ordonnée à l'origine de la droite de régression |
| Score (console) | Coefficient de détermination R² de la régression (proche de 1 si la croissance est linéaire) |
| `puissance(a, V)` (console) | Puissance de fusion extrapolée (W) |
| Graphique | Nombre de fusions en fonction du temps, avec la droite de régression |