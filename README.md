# Rapport de bug — Convoyeur bi-directionnel

| | |
|---|---|
| **Logiciel** | Factory I/O v2.5.10 + Control I/O |
| **Fichier** | `Diagram1.controlio` |
| **Statut** | ✅ Résolu & Validé |

---

## ❌ Le problème

Quand on lançait le carton sur le convoyeur, **le tapis ne s'arrêtait pas en fin de course**.

Le carton continuait de forcer jusqu'au bout du tapis et tombait par terre. Et une fois en bout de course, impossible de le faire repartir dans l'autre sens sans qu'il tombe de l'autre côté.

---

## 🔍 La cause

Dans Control I/O, les deux boutons de commande (gauche et droite) étaient branchés sur le **même canal moteur**.

Résultat : quand un capteur détectait le carton et tentait de couper le moteur, le signal de l'autre bouton maintenait le moteur actif. **Le tapis ne s'arrêtait jamais.**

---

## 🛠️ La correction

### Étape 1 — Séparer les circuits

Le convoyeur a été configuré en mode **Digital ±** (bi-directionnel) dans Factory I/O.

Chaque bouton a ensuite été branché sur son **propre canal moteur** avec son **propre capteur de sécurité** :

| Bouton | Capteur d'arrêt | Canal moteur | Direction |
|:---|:---|:---|:---|
| Start Button 0 | Diffuse Sensor 0 | Belt Conveyor 0 `(-)` | ⬅️ Gauche |
| Start Button 2 | Diffuse Sensor 1 | Belt Conveyor 0 `(+)` | ➡️ Droite |

La logique de sécurité appliquée sur chaque circuit :

```
Bouton ──────────────┐
                     ├──► [ AND ] ──► Moteur
Capteur ──► [ NOT ] ─┘
```

> Dès que le carton coupe le faisceau du capteur → `NOT` passe à `0` → `AND` coupe le moteur instantanément, **même si le bouton est encore enfoncé**.

### Étape 2 — Ajustement ergonomique

Après correction, les boutons physiques du pupitre ont été **permutés** pour que le bouton gauche envoie bien le carton à gauche, et le bouton droit à droite.

> ⚠️ Ce n'est pas une correction de bug — c'est un ajustement de confort pour l'opérateur.

---

## ✅ Résultat

- Le carton s'arrête tout seul, pile sur le capteur en fin de course
- On peut faire des aller-retours à l'infini sans risque de chute
- Même bouton maintenu appuyé → le capteur reste prioritaire et coupe le moteur
- Les boutons correspondent au sens de déplacement physique du carton

---

## 📎 Fichiers joints

| Fichier | Description |
|:---|:---|
| `ERREUR_INVERSION_PROJET_1.mp4` | Vidéo du bug avant correction |
| `Capture_d_écran_ERREUR_INVERSION.png` | Diagramme Control I/O — état bugué |
| `Capture_d_écran_2026-05-19.png` | Diagramme Control I/O — état corrigé |
