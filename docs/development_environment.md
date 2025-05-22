# CertForge - Configuration de l'Environnement de Développement

## Vue d'ensemble

Ce document décrit la configuration de l'environnement de développement pour le projet CertForge, une application Android de préparation aux certifications professionnelles. Il couvre les outils, les versions, les conventions et les processus CI/CD nécessaires pour contribuer efficacement au projet.

## Prérequis

### Logiciels requis

- **Android Studio** : Version Arctic Fox (2021.3.1) ou supérieure
- **JDK** : Version 11 ou supérieure
- **Git** : Version 2.30.0 ou supérieure
- **Gradle** : Version 7.2.0 ou supérieure (inclus dans Android Studio)

### Comptes et accès

- Compte GitHub pour l'accès au dépôt
- Compte Google Cloud Console pour les API Google Drive
- Accès au projet Google Cloud pour les clés API

## Configuration du projet

### Clonage du dépôt

```bash
git clone https://github.com/certforge/certforge-android.git
cd certforge-android
```

### Configuration d'Android Studio

1. Ouvrir Android Studio
2. Sélectionner "Open an existing project"
3. Naviguer vers le dossier cloné et sélectionner le fichier `build.gradle` à la racine
4. Laisser Gradle synchroniser le projet

### Configuration des clés API

1. Créer un fichier `local.properties` à la racine du projet (non versionné)
2. Ajouter les clés API nécessaires :

```properties
# Google Drive API
GOOGLE_DRIVE_CLIENT_ID=your_client_id_here
GOOGLE_DRIVE_CLIENT_SECRET=your_client_secret_here

# Autres clés API si nécessaire
```

### Configuration des variables d'environnement

Pour les builds CI/CD, configurer les variables d'environnement suivantes dans votre système CI :

- `GOOGLE_DRIVE_CLIENT_ID`
- `GOOGLE_DRIVE_CLIENT_SECRET`
- `KEYSTORE_PASSWORD`
- `KEY_ALIAS`
- `KEY_PASSWORD`

## Structure du projet

```
certforge-android/
├── app/                    # Module principal de l'application
├── core/                   # Module core avec composants partagés
├── domain/                 # Module domain avec entités et use cases
├── data/                   # Module data avec repositories et sources
├── feature/                # Modules de fonctionnalités
│   ├── auth/               # Fonctionnalité d'authentification
│   ├── deckmanagement/     # Gestion des decks de flashcards
│   ├── quiz/               # Fonctionnalité de quiz
│   └── statistics/         # Statistiques et suivi
├── buildSrc/               # Configuration Gradle partagée
├── gradle/                 # Scripts et configurations Gradle
├── docs/                   # Documentation du projet
└── scripts/                # Scripts utilitaires
```

## Conventions de code

### Style de code Kotlin

Le projet suit les conventions de style Kotlin officielles avec quelques spécificités :

- Indentation : 4 espaces
- Longueur maximale de ligne : 120 caractères
- Nommage des classes : PascalCase
- Nommage des fonctions et variables : camelCase
- Constantes : SNAKE_CASE_MAJUSCULE

### Architecture

- Pattern MVVM pour la couche présentation
- Clean Architecture pour la séparation des couches
- Injection de dépendances avec Hilt
- Flux de données unidirectionnel avec StateFlow/SharedFlow

### Commentaires et documentation

- Commentaires KDoc pour les classes et fonctions publiques
- Commentaires explicatifs pour la logique complexe
- Éviter les commentaires redondants avec le code
- Documentation des décisions d'architecture dans des fichiers dédiés

## Outils de qualité de code

### Analyse statique

- **Ktlint** : Vérification du style de code Kotlin
- **Detekt** : Analyse statique avancée pour Kotlin
- **Android Lint** : Vérification spécifique à Android

Configuration dans le fichier `quality.gradle` :

```groovy
ktlint {
    version = "0.45.2"
    android = true
    reporters {
        reporter "plain"
        reporter "checkstyle"
    }
    filter {
        exclude("**/generated/**")
    }
}

detekt {
    config = files("$rootDir/config/detekt/detekt.yml")
    reports {
        html.enabled = true
        xml.enabled = true
    }
}
```

### Tests

- **JUnit 5** : Tests unitaires
- **Mockk** : Framework de mock pour Kotlin
- **Espresso** : Tests UI Android
- **Robolectric** : Tests Android sans émulateur

Configuration dans le fichier `testing.gradle` :

```groovy
dependencies {
    testImplementation "org.junit.jupiter:junit-jupiter:5.8.2"
    testImplementation "io.mockk:mockk:1.12.0"
    androidTestImplementation "androidx.test.espresso:espresso-core:3.4.0"
    testImplementation "org.robolectric:robolectric:4.7.3"
}
```

## Configuration CI/CD

### GitHub Actions

Le projet utilise GitHub Actions pour l'intégration continue et le déploiement continu. Les workflows principaux sont :

#### CI Workflow (.github/workflows/ci.yml)

```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'adopt'
          
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew
        
      - name: Run ktlint
        run: ./gradlew ktlintCheck
        
      - name: Run detekt
        run: ./gradlew detekt
        
      - name: Run unit tests
        run: ./gradlew testDebugUnitTest
        
      - name: Build debug APK
        run: ./gradlew assembleDebug
        
      - name: Upload APK
        uses: actions/upload-artifact@v2
        with:
          name: app-debug
          path: app/build/outputs/apk/debug/app-debug.apk
```

#### Release Workflow (.github/workflows/release.yml)

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'adopt'
          
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew
        
      - name: Decode Keystore
        env:
          ENCODED_KEYSTORE: ${{ secrets.KEYSTORE_BASE64 }}
        run: |
          echo $ENCODED_KEYSTORE | base64 -d > keystore.jks
          
      - name: Build release APK
        env:
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
        run: ./gradlew assembleRelease
        
      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          files: app/build/outputs/apk/release/app-release.apk
          draft: false
          prerelease: false
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Firebase App Distribution

Pour la distribution des builds de test :

```yaml
- name: Upload to Firebase App Distribution
  uses: wzieba/Firebase-Distribution-Github-Action@v1
  with:
    appId: ${{ secrets.FIREBASE_APP_ID }}
    token: ${{ secrets.FIREBASE_TOKEN }}
    groups: testers
    file: app/build/outputs/apk/debug/app-debug.apk
    releaseNotes: |
      Changes in this build:
      - ${{ github.event.head_commit.message }}
```

## Gestion des dépendances

### Versions centralisées

Les versions des dépendances sont centralisées dans le fichier `buildSrc/src/main/kotlin/Dependencies.kt` :

```kotlin
object Versions {
    const val kotlin = "1.6.10"
    const val compose = "1.2.0-alpha08"
    const val hilt = "2.41"
    const val room = "2.4.2"
    // Autres versions...
}

object Deps {
    const val kotlinStdLib = "org.jetbrains.kotlin:kotlin-stdlib:${Versions.kotlin}"
    
    object AndroidX {
        const val core = "androidx.core:core-ktx:1.7.0"
        const val appCompat = "androidx.appcompat:appcompat:1.4.1"
        // Autres dépendances AndroidX...
    }
    
    object Compose {
        const val ui = "androidx.compose.ui:ui:${Versions.compose}"
        const val material = "androidx.compose.material:material:${Versions.compose}"
        const val tooling = "androidx.compose.ui:ui-tooling:${Versions.compose}"
        // Autres dépendances Compose...
    }
    
    // Autres groupes de dépendances...
}
```

### Utilisation dans les modules

```kotlin
dependencies {
    implementation(Deps.kotlinStdLib)
    implementation(Deps.AndroidX.core)
    implementation(Deps.Compose.ui)
    implementation(Deps.Compose.material)
    // Autres dépendances...
}
```

## Environnements de build

### Types de build

- **Debug** : Pour le développement quotidien
- **Staging** : Pour les tests internes
- **Release** : Pour la production

Configuration dans `app/build.gradle.kts` :

```kotlin
android {
    buildTypes {
        debug {
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-DEBUG"
            isDebuggable = true
        }
        
        create("staging") {
            applicationIdSuffix = ".staging"
            versionNameSuffix = "-STAGING"
            isDebuggable = true
            signingConfig = signingConfigs.getByName("debug")
        }
        
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
            signingConfig = signingConfigs.getByName("release")
        }
    }
}
```

### Variantes de produit

Pour gérer différentes saveurs de l'application :

```kotlin
android {
    flavorDimensions("version")
    
    productFlavors {
        create("free") {
            dimension = "version"
            applicationIdSuffix = ".free"
            versionNameSuffix = "-free"
        }
        
        create("premium") {
            dimension = "version"
            applicationIdSuffix = ".premium"
            versionNameSuffix = "-premium"
        }
    }
}
```

## Démarrage rapide pour les développeurs

### Première configuration

1. Cloner le dépôt
2. Ouvrir le projet dans Android Studio
3. Copier `local.properties.example` vers `local.properties` et remplir les clés API
4. Exécuter `./gradlew build` pour vérifier que tout fonctionne

### Workflow de développement

1. Créer une branche à partir de `develop` : `git checkout -b feature/ma-fonctionnalite`
2. Implémenter la fonctionnalité
3. Exécuter les vérifications locales : `./gradlew check`
4. Soumettre une Pull Request vers `develop`
5. Attendre la validation CI et la revue de code
6. Merger la PR une fois approuvée

### Commandes utiles

```bash
# Construire l'application
./gradlew assembleDebug

# Exécuter les tests unitaires
./gradlew testDebugUnitTest

# Vérifier le style de code
./gradlew ktlintCheck

# Analyse statique
./gradlew detekt

# Installer sur un appareil connecté
./gradlew installDebug

# Générer la documentation
./gradlew dokkaHtml
```

## Ressources et documentation

### Documentation interne

- Architecture : `/docs/architecture.md`
- Guide de style : `/docs/style-guide.md`
- Processus de contribution : `/docs/contributing.md`

### Ressources externes

- [Documentation Kotlin](https://kotlinlang.org/docs/home.html)
- [Documentation Android](https://developer.android.com/docs)
- [Documentation Jetpack Compose](https://developer.android.com/jetpack/compose/documentation)
- [Documentation Google Drive API](https://developers.google.com/drive/api/v3/about-sdk)

## Dépannage

### Problèmes courants

#### Erreur de synchronisation Gradle

```
Invalidate caches / Restart dans Android Studio
```

#### Problèmes avec les clés API

```
Vérifier que les clés sont correctement configurées dans local.properties
```

#### Erreurs de compilation Compose

```
Vérifier la compatibilité des versions de Compose et du plugin Kotlin
```

### Support

Pour toute question ou problème :

- Ouvrir une issue sur GitHub
- Contacter l'équipe de développement via Slack
- Consulter la documentation interne
