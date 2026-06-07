# Introduction aux LLMs — démos interactives

Quatre widgets pédagogiques regroupés dans une seule application à onglets, pour accompagner un cours
d'introduction aux grands modèles de langage (LLMs). L'arc va du modèle le plus simple (compter des
paires de mots) jusqu'au mécanisme d'attention des transformers.

## Onglets

1. **Bigramme** — modèle bigramme entraîné en direct sur un corpus au choix (Candide de Voltaire en
   français, ou *Dr Jekyll & Mr Hyde* de Stevenson en anglais, ou un texte collé librement).
2. **Embeddings** — des tokens bruts aux vecteurs sémantiques.
3. **Analogies** — arithmétique vectorielle (roi − homme + femme ≈ reine), visualisée avec Chart.js.
4. **Attention** — limite du bigramme, mécanisme d'attention, Q/K/V, multi-tête.

## Utilisation

Application 100 % statique : ouvrir `index.html` dans un navigateur, ou la consulter en ligne via
GitHub Pages. Chaque widget est isolé dans une iframe ; l'onglet « Analogies » charge Chart.js depuis
un CDN (connexion requise).

## Crédits

Textes de corpus issus du domaine public (Project Gutenberg). Widgets générés avec Claude.
