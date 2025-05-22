# CertForge - Architecture Technique

## Vue d'ensemble

CertForge est une application Android native développée en Kotlin qui permet aux utilisateurs de créer, organiser et étudier des flashcards pour la préparation aux certifications professionnelles. L'application utilise une architecture MVVM (Model-View-ViewModel) combinée avec les principes de Clean Architecture pour garantir une séparation claire des responsabilités, une testabilité optimale et une évolutivité à long terme.

## Principes architecturaux

### Clean Architecture

L'application est structurée en couches concentriques selon les principes de Clean Architecture :

1. **Couche Domain** (cœur de l'application)
   - Entités métier
   - Cas d'utilisation (Use Cases)
   - Interfaces des repositories

2. **Couche Data**
   - Implémentations des repositories
   - Sources de données (locales et distantes)
   - Mappers pour la conversion entre modèles de données

3. **Couche Presentation**
   - ViewModels
   - UI States
   - Composants Jetpack Compose

4. **Couche Framework**
   - Implémentations spécifiques à Android
   - Services système
   - Intégrations tierces (Google Drive API)

### MVVM (Model-View-ViewModel)

Le pattern MVVM est utilisé pour la couche présentation :

- **Model** : Représente les données et la logique métier (Use Cases et Repositories)
- **View** : Composants Jetpack Compose qui affichent l'interface utilisateur
- **ViewModel** : Fait le lien entre le Model et la View, gère l'état de l'UI et traite les événements utilisateur

### Flux de données unidirectionnel (UDF)

L'application implémente un flux de données unidirectionnel pour une gestion d'état prévisible :

1. L'utilisateur interagit avec l'UI
2. Les événements sont envoyés au ViewModel
3. Le ViewModel traite les événements et met à jour l'état
4. L'UI observe l'état et se met à jour en conséquence

## Structure des modules

L'application est organisée en modules pour favoriser la séparation des préoccupations :

### Module App

Module principal qui intègre tous les autres modules et contient la configuration de l'application.

### Module Core

Contient les composants partagés entre tous les modules :
- Utilitaires communs
- Extensions Kotlin
- Composants UI réutilisables
- Navigation
- Injection de dépendances

### Module Domain

Contient la logique métier indépendante de tout framework :
- Entités métier (Flashcard, Deck, Category, etc.)
- Use Cases
- Interfaces des repositories

### Module Data

Gère l'accès aux données et leur persistance :
- Implémentations des repositories
- Sources de données locales (Room)
- Sources de données distantes (Google Drive API)
- Mappers pour la conversion entre modèles

### Modules Feature

Modules organisés par fonctionnalité :
- **Feature:Auth** : Authentification Google
- **Feature:DeckManagement** : Gestion des jeux de flashcards
- **Feature:Quiz** : Sessions d'étude et quiz
- **Feature:Statistics** : Suivi des performances et statistiques

## Composants techniques clés

### Persistence des données

#### Room Database

- Stockage local des flashcards, decks, et statistiques utilisateur
- Entités Room mappées aux entités de domaine
- DAOs (Data Access Objects) pour les opérations CRUD
- Migrations pour les évolutions de schéma

#### DataStore

- Stockage des préférences utilisateur
- Configuration de l'application
- Tokens d'authentification

### Intégration Google Drive

#### Google Drive API

- Authentification OAuth 2.0
- Lecture/écriture des fichiers YAML/JSON
- Surveillance des modifications
- Gestion des conflits de synchronisation

#### WorkManager

- Synchronisation périodique en arrière-plan
- Téléchargement des fichiers pour utilisation hors ligne
- Envoi des modifications locales vers Google Drive

### Parseur YAML/JSON

- Utilisation de bibliothèques Jackson/Gson pour JSON
- Intégration de SnakeYAML pour YAML
- Validation de schéma
- Gestion des erreurs et feedback utilisateur

### Interface utilisateur

#### Jetpack Compose

- UI déclarative et réactive
- Composants personnalisés pour les flashcards et QCM
- Thèmes (clair/sombre) et styles cohérents
- Animations et transitions

#### Navigation Compose

- Navigation entre les écrans
- Arguments de navigation typés
- Deep linking pour les notifications

### Injection de dépendances

#### Hilt

- Injection de dépendances à l'échelle de l'application
- Modules par fonctionnalité
- Scopes appropriés (Singleton, Activity, ViewModel)

## Diagrammes

### Diagramme d'architecture globale

```
┌─────────────────────────────────────────────────────────────┐
│                        UI Layer                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Screens    │    │  Composants │    │   Thèmes    │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                     Presentation Layer                      │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │  ViewModels │    │  UI States  │    │   Events    │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                      Domain Layer                           │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Entities  │    │  Use Cases  │    │ Repositories│     │
│  │             │    │             │    │ (Interfaces)│     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                       Data Layer                            │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │ Repositories│    │Data Sources │    │   Mappers   │     │
│  │(Impl)       │    │             │    │             │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                    Framework Layer                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │    Room     │    │ Google Drive│    │ WorkManager │     │
│  │  Database   │    │     API     │    │             │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### Diagramme de flux de données

```
┌──────────────┐     ┌───────────────┐     ┌───────────────┐
│  Google Drive│     │  Local Files  │     │  User Input   │
└──────┬───────┘     └───────┬───────┘     └───────┬───────┘
       │                     │                     │
       ▼                     ▼                     ▼
┌──────────────────────────────────────────────────────────┐
│                    Data Sources                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │
│  │ Drive API   │    │File Parsers │    │ Room DAOs   │  │
│  └─────────────┘    └─────────────┘    └─────────────┘  │
└──────────────────────────────┬───────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────┐
│                    Repositories                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │
│  │FlashcardRepo│    │CategoryRepo │    │ StatsRepo   │  │
│  └─────────────┘    └─────────────┘    └─────────────┘  │
└──────────────────────────────┬───────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────┐
│                     Use Cases                            │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │
│  │GetFlashcards│    │StartQuizSess│    │TrackProgress│  │
│  └─────────────┘    └─────────────┘    └─────────────┘  │
└──────────────────────────────┬───────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────┐
│                    ViewModels                            │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │
│  │  QuizVM     │    │ DeckListVM  │    │  StatsVM    │  │
│  └─────────────┘    └─────────────┘    └─────────────┘  │
└──────────────────────────────┬───────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────┐
│                  Compose UI                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │
│  │ QuizScreen  │    │DeckListScrn │    │ StatsScreen │  │
│  └─────────────┘    └─────────────┘    └─────────────┘  │
└──────────────────────────────────────────────────────────┘
```

## Gestion des états et des erreurs

### États UI

Chaque ViewModel expose un état UI immuable qui représente l'état actuel de l'écran :

```kotlin
data class QuizScreenState(
    val isLoading: Boolean = false,
    val currentFlashcard: FlashcardUiModel? = null,
    val progress: Float = 0f,
    val error: String? = null,
    val isOfflineMode: Boolean = false
)
```

### Gestion des erreurs

- Utilisation de `Result<T>` pour encapsuler les succès et les échecs
- Centralisation de la gestion des erreurs dans les ViewModels
- Affichage contextuel des erreurs dans l'UI
- Stratégies de retry pour les opérations réseau

### Coroutines et Flow

- Utilisation de Coroutines pour les opérations asynchrones
- Flow pour les flux de données réactifs
- StateFlow pour l'état observable des ViewModels
- SharedFlow pour les événements ponctuels (navigation, toasts)

## Sécurité et confidentialité

### Authentification

- OAuth 2.0 pour l'authentification Google
- Stockage sécurisé des tokens d'accès
- Gestion des expirations et rafraîchissement automatique

### Stockage local

- Chiffrement de la base de données Room
- Isolation des données utilisateur
- Pas de stockage de données sensibles en clair

### Permissions

- Demande minimale de permissions Android
- Justification claire pour chaque permission
- Gestion gracieuse des refus de permissions

## Tests

### Tests unitaires

- Tests des Use Cases
- Tests des ViewModels
- Tests des Repositories
- Mocks pour les dépendances externes

### Tests d'intégration

- Tests des flux de données complets
- Tests de l'intégration Room
- Tests de l'intégration Google Drive

### Tests UI

- Tests des composants Compose
- Tests de navigation
- Tests d'accessibilité

## Performance et optimisation

### Chargement différé

- Chargement paresseux des données
- Pagination pour les grandes listes
- Préchargement intelligent basé sur les habitudes utilisateur

### Mise en cache

- Stratégie de mise en cache à plusieurs niveaux
- Invalidation intelligente du cache
- Priorité aux données locales pour une expérience réactive

### Mode hors ligne

- Fonctionnement complet sans connexion
- Synchronisation en arrière-plan lors de la reconnexion
- Indicateurs clairs de l'état de synchronisation

## Évolutivité

L'architecture est conçue pour faciliter l'ajout de nouvelles fonctionnalités :

- Nouveaux types de questions
- Intégrations supplémentaires (Obsidian, etc.)
- Fonctionnalités premium futures
- Support multi-plateforme potentiel (via Kotlin Multiplatform)
