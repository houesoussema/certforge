# CertForge - Système de Design UI

## Principes de design

CertForge suit une approche de design centrée sur l'utilisateur avec les principes suivants :

1. **Simplicité** - Interface épurée et intuitive pour une utilisation rapide
2. **Accessibilité** - Éléments suffisamment grands et contrastés pour une utilisation mobile
3. **Cohérence** - Patterns d'interaction uniformes à travers l'application
4. **Feedback** - Retour visuel clair pour toutes les actions utilisateur
5. **Efficacité** - Optimisé pour des sessions d'étude courtes et productives

## Palette de couleurs

### Couleurs primaires
- **Primaire** : `#2962FF` (Bleu intense)
- **Primaire variante** : `#0039CB` (Bleu foncé)
- **Secondaire** : `#00BFA5` (Turquoise)
- **Secondaire variante** : `#008E76` (Turquoise foncé)

### Couleurs fonctionnelles
- **Succès** : `#4CAF50` (Vert)
- **Erreur** : `#F44336` (Rouge)
- **Avertissement** : `#FF9800` (Orange)
- **Information** : `#2196F3` (Bleu clair)

### Couleurs neutres
- **Fond clair** : `#FFFFFF` (Blanc)
- **Fond sombre** : `#121212` (Noir presque pur)
- **Surface claire** : `#F5F5F5` (Gris très clair)
- **Surface sombre** : `#1E1E1E` (Gris très foncé)
- **Texte sur clair** : `#212121` (Gris très foncé)
- **Texte sur sombre** : `#EEEEEE` (Gris très clair)

## Typographie

### Familles de polices
- **Titre** : Roboto Medium
- **Corps** : Roboto Regular
- **Accentuation** : Roboto Bold
- **Monospace** (pour code) : JetBrains Mono

### Échelle typographique
- **Titre principal (H1)** : 24sp
- **Titre secondaire (H2)** : 20sp
- **Titre tertiaire (H3)** : 18sp
- **Corps de texte** : 16sp
- **Texte secondaire** : 14sp
- **Légende** : 12sp

## Composants UI

### Cartes (Cards)
- **Flashcard standard** : Carte avec question au recto, réponse au verso
- **Carte QCM** : Carte avec question et options de réponse
- **Carte de catégorie** : Carte représentant une catégorie avec compteur
- **Carte de statistique** : Carte affichant une métrique de performance

### Boutons
- **Bouton primaire** : Action principale, couleur primaire
- **Bouton secondaire** : Action secondaire, contour de couleur primaire
- **Bouton tertiaire** : Action tertiaire, texte de couleur primaire
- **Bouton flottant (FAB)** : Action principale contextuelle

### Champs de saisie
- **Champ de texte standard** : Pour la saisie de texte simple
- **Champ de recherche** : Avec icône de recherche et effacement
- **Sélecteur** : Pour choisir parmi des options prédéfinies

### Navigation
- **Barre inférieure** : Navigation principale entre sections
- **Barre supérieure** : Titre de l'écran et actions contextuelles
- **Tiroir de navigation** : Menu latéral pour options supplémentaires
- **Onglets** : Pour basculer entre sous-sections

### Feedback
- **Toast** : Message éphémère pour confirmation
- **Snackbar** : Message avec action possible
- **Dialogue** : Pour les confirmations importantes
- **Indicateur de progression** : Pour les opérations longues

## Iconographie

CertForge utilise le set d'icônes Material Design avec une emphase sur la clarté et la reconnaissance. Les icônes principales incluent :

- **Accueil** : Maison
- **Bibliothèque** : Pile de cartes
- **Quiz** : Point d'interrogation dans un cercle
- **Statistiques** : Graphique en barres
- **Paramètres** : Engrenage
- **Synchronisation** : Flèches circulaires
- **Marquer** : Signet
- **Filtrer** : Entonnoir

## Animations et transitions

### Animations de cartes
- **Retournement** : Animation 3D pour révéler la réponse
- **Balayage** : Pour passer à la carte suivante
- **Expansion** : Pour afficher plus de détails

### Transitions d'écran
- **Fondu enchaîné** : Transition standard entre écrans
- **Glissement** : Pour la navigation hiérarchique
- **Expansion partagée** : Pour les transitions détaillées

### Animations de feedback
- **Pulsation** : Pour indiquer une action possible
- **Ondulation** : Pour le feedback tactile
- **Progression** : Pour les opérations asynchrones

## Modes d'affichage

### Mode clair
- Fond blanc
- Surfaces légèrement grisées
- Texte foncé
- Accents de couleur primaire

### Mode sombre
- Fond noir profond
- Surfaces gris foncé
- Texte clair
- Accents de couleur primaire légèrement désaturés

### Mode économie d'énergie
- Fond noir pur
- Interface minimaliste
- Animations réduites
- Optimisé pour OLED

## Accessibilité

### Contraste
- Ratio de contraste minimum de 4.5:1 pour tout texte
- Indicateurs visuels non basés uniquement sur la couleur

### Taille des cibles
- Taille minimale de 48dp pour tous les éléments interactifs
- Espacement adéquat pour éviter les erreurs de toucher

### Alternatives textuelles
- Descriptions pour toutes les images
- Labels pour tous les contrôles interactifs

### Navigation
- Support complet du lecteur d'écran
- Navigation possible au clavier
- Structure hiérarchique claire

## Responsive Design

### Téléphones
- Layout à une colonne
- Navigation en bas
- Éléments optimisés pour utilisation à une main

### Tablettes
- Layout à deux colonnes
- Navigation latérale possible
- Utilisation efficace de l'espace supplémentaire

### Orientation
- Support portrait et paysage
- Réorganisation intelligente des éléments
- Préservation du contexte lors des changements d'orientation
