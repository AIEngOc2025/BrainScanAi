# Rapport de Synthèse : Mission BrainScanAI

## 1. Résumé de la Mission
L'objectif de cette mission était d'explorer et de modéliser un dataset de radiographies cérébrales (MRI) pour détecter la présence de cancer, en utilisant une approche hybride (non supervisée et semi-supervisée) compte tenu de la rareté des labels experts.

## 2. Approche Methodologique

### Phase 1 : Audit et Exploration
- **Volume** : 1506 images au total.
- **Déséquilibre** : Seulement 100 images labellisées (50 Normal / 50 Cancer) contre 1406 images sans label.
- **Qualité** : Images PNG de dimensions variables, converties en tenseurs 224x224 pour le traitement.

### Phase 2 : Extraction de Caractéristiques
- **Modèle** : ResNet50 (pré-entraîné sur ImageNet) utilisé comme extracteur de caractéristiques (feature extractor).
- **Sortie** : Chaque image est représentée par un vecteur de **2048 dimensions**.
- **Stockage** : Embeddings sauvegardés dans `data/features_brainscan.npy`.

### Phase 3 : Analyse Non-Supervisée
- **Technique** : Réduction de dimension (PCA + t-SNE) et Clustering (K-Means).
- **Résultat** : Les regroupements naturels montrent une séparation partielle entre les cas sains et pathologiques, validant la pertinence des caractéristiques extraites par ResNet50.

### Phase 4 : Modélisation Semi-Supervisée
- **Technique** : **Pseudo-Labeling (Self-Training)** avec Régression Logistique.
- **Objectif** : Maximiser le **Recall** (sensibilité) pour la détection du cancer (priorité médicale).
- **Performance** : L'utilisation des 1406 images non labellisées a permis d'affiner les frontières de décision du modèle initialement entraîné sur seulement 80 échantillons (train set).

## 3. Recommandations Stratégiques pour le Passage à l'Échelle

### Objectif : Traitement de 4 millions d'images
**Budget cible : 5 000 €**

#### Étude de faisabilité :
1. **Labellisation par des experts** (Radiologues) :
   - Coût estimé : 3€ par image (selon le budget initial de 300€ pour 100 images).
   - Pour 4M d'images, le coût serait de 12M€, ce qui est hors budget.
2. **Stratégie proposée : Active Learning + Semi-Supervised** :
   - Utiliser l'approche développée (Pseudo-labeling) pour pré-annoter les 4M d'images.
   - Utiliser le budget de 5 000 € pour faire vérifier par des experts les cas où le modèle a la plus faible confiance (**Uncertainty Sampling**).
   - À 3€ par image, 5 000 € permettent d'annoter environ **1 600 nouvelles images stratégiques**.

#### Architecture Technique :
- Passage sur Infrastructure Cloud (AWS Sagemaker ou Azure ML).
- Utilisation de GPU (NVIDIA A100/H100) pour l'extraction de caractéristiques en batch sur 4M d'images.
- Stockage optimisé (Parquet ou TFRecord) pour la lecture rapide des caractéristiques.

## 4. Conclusion
L'approche semi-supervisée est la seule viable pour ce projet. Le pipeline actuel est prêt pour un déploiement "Proof of Concept" (PoC) et peut être étendu massivement en couplant le pseudo-labeling automatique à une vérification humaine ciblée.
