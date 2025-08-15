Modélisation de l'Évolution Tumorale par Imagerie IRM 3D
Projet de Master Big Data, UCAO-UUTPrésenté par : KOMOSSI Sosso (komossisosso@gmail.com)Superviseur : M. Sena Apeke
Aperçu du Projet
Ce projet propose un pipeline intégré pour la segmentation et la prédiction de l’évolution spatio-temporelle des tumeurs cérébrales à partir d’images IRM volumiques (format NIfTI). Il combine deux approches complémentaires :

Segmentation par diffusion : Inspirée des avancées récentes en imagerie médicale, utilisant un modèle de diffusion (DDPM) pour une segmentation robuste avec estimation des incertitudes.
Prévision spatio-temporelle : Un AutoEncodeur 3D débruitant couplé à un ConvLSTM 3D pour prédire l’état futur (t3) à partir d’observations passées (t1, t2), avec une option de segmentation.

Le pipeline est modulaire, reproductible et compatible CPU/GPU. Il inclut un script unifié (main.py) pour l’entraînement, l’inférence et l’évaluation, un tableau de bord interactif (Streamlit) pour la visualisation, ainsi qu’un script de bootstrap pour générer des poids initiaux et journaux de pertes.
Lien vers l’application : Tumor Evolution Dashboard
Objectifs

Détection précoce : Identifier les tumeurs cérébrales via une segmentation précise.
Suivi thérapeutique : Évaluer la réponse au traitement à partir de données IRM multi-temporelles.
Prévision d’évolution : Anticiper l’état futur des tumeurs pour personnaliser les traitements.
Accessibilité : Fournir un outil convivial avec interface graphique et compatibilité CPU/GPU.

Structure du Projet
├── main.py                    # Script principal (entraînement, inférence, évaluation)
├── sanity_check.py           # Vérification du mappage t1/t2/t3
├── quick_bootstrap_ckpts.py  # Génération rapide des poids et journaux
├── app_singlecol_phys.py     # Tableau de bord Streamlit
├── requirements.txt          # Dépendances
└── .idea/runConfigurations   # Configurations PyCharm

Technologies clés : PyTorch, MONAI, Nibabel, Streamlit, NumPy, Pandas, Matplotlib, TQDM.
Prérequis

Python 3.8+
Dépendances listées dans requirements.txt :pip install -r requirements.txt


Pour GPU NVIDIA : Installez une version PyTorch compatible CUDA (ex. cu121).
Jeu de données : IRM FLAIR longitudinales (ex. Yale-Brain-Mets-Longitudinal), structuré comme suit :<root>/<PatientID>/<YYYY-MM-DD>/<PatientID>_<YYYY-MM-DD>_<HH-MM-SS>_FLAIR.nii.gz



Installation

Clonez le dépôt :git clone <url-du-dépôt>
cd <nom-du-dépôt>


Installez les dépendances :pip install -r requirements.txt


Configurez le chemin des données via --data_root dans les commandes.

Utilisation
1. Vérification des données
Vérifiez le mappage des séries t1/t2/t3 :
python sanity_check.py --scan_nested --modality FLAIR --data_root "<chemin-vers-donnees>"

2. Entraînement

Prévision (AutoEncodeur + ConvLSTM) :python main.py --scan_nested --modality FLAIR \
  --data_root "<chemin-vers-donnees>" \
  --train_forecast --forecast_epochs 10 --batch_size 1 \
  --spacing 1.5 1.5 1.5 --cache_rate 0.1 --multitask_seg


Segmentation par diffusion (si masques disponibles) :python main.py --scan_nested --modality FLAIR \
  --data_root "<chemin-vers-donnees>" \
  --train_diffusion --diff_epochs 10 --batch_size 1



3. Inférence et évaluation
python main.py --scan_nested --modality FLAIR --data_root "<chemin-vers-donnees>" --infer

4. Bootstrap des poids
Générez des poids initiaux pour un démarrage rapide :
python quick_bootstrap_ckpts.py --scan_nested --modality FLAIR \
  --data_root "<chemin-vers-donnees>" --out_dir out \
  --spacing 2.0 --cache_rate 0.05 --steps_forecast 30 --steps_diff 20

Sorties : out/forecast.pt, out/diffusion_seg.pt, out/forecast_log.csv, out/diffusion_log.csv.
5. Tableau de bord Streamlit
Lancez l’interface interactive :
streamlit run app_singlecol_phys.py

Fonctionnalités :

Importation de 1 à 3 volumes NIfTI (t1, t2, t3).
Visualisation 2D (axial, coronal, sagittal) à l’échelle physique (mm).
Statistiques descriptives et histogrammes.
Prétraitements avec aperçu des dimensions.
Prédictions (t3), cartes d’erreur, segmentations par diffusion.
Courbes de pertes basées sur les journaux CSV.

Résultats

Prévision : Courbe de perte décroissante sur 30 étapes (out/forecast_log.csv).
Segmentation : Perte MSE(ε) décroissante sur ~20 étapes, ou journal factice si masques absents.
Tableau de bord : Affichage précis des coupes, cartes de chaleur et erreurs (|t3_pred−t3|).
Pour une évaluation rigoureuse, prolongez l’entraînement (10–50 époques) et validez sur un jeu de test figé.

Reproductibilité

Environnement : Utilisez requirements.txt pour un environnement cohérent.
Données : Stockez les datasets sur un disque dédié (40+ Go). Pointez --data_root.
Sorties : Journaux et fichiers générés dans out/.
PyCharm : Configurations prêtes pour entraînement, inférence, vérification et Streamlit.

Limites et Améliorations

Masques : L’absence de vérité terrain limite la segmentation. Envisager des annotations partielles ou pseudo-étiquetage.
Anisotropie : Tester un rééchantillonnage 1×1×1 mm si mémoire suffisante.
Modèles : Explorer UNet/UNETR 3D, SwinUNETR ou Transformers temporels.
Incertitude : Quantifier via MC Dropout ou échantillonnages multiples.
Multi-modalités : Intégrer FLAIR, T1, T1CE, T2.
Post-traitement : Appliquer des opérations morphologiques ou CRF.

Références

Ambiguous Medical Image Segmentation using Diffusion Models.
MedSegDiff-V2: Diffusion-Based Medical Image Segmentation with Transformer.
Prediction of Ocean Weather Based on Denoising AutoEncoder and Convolutional LSTM.

Contact
Pour toute question, contactez :KOMOSSI Sosso – komossisosso@gmail.com
