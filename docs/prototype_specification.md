# CertForge - Spécification du Prototype Fonctionnel

## Objectifs du prototype

Le prototype fonctionnel minimal de CertForge vise à valider les concepts fondamentaux de l'application et à démontrer la faisabilité technique des fonctionnalités clés. Ce prototype se concentrera sur :

1. L'authentification Google et l'intégration avec Google Drive
2. Le parseur YAML/JSON pour les flashcards et QCM
3. Une interface utilisateur basique pour la navigation et l'affichage des flashcards
4. Un moteur de quiz simple pour tester l'apprentissage

## Portée du prototype

### Inclus dans le prototype

- Authentification OAuth 2.0 avec Google
- Lecture des fichiers YAML/JSON depuis Google Drive
- Parseur pour les formats de flashcards standard et QCM
- Navigation entre les écrans principaux
- Affichage des flashcards et QCM
- Session de quiz basique
- Stockage local minimal pour le mode hors ligne

### Exclus du prototype

- Synchronisation bidirectionnelle complète
- Statistiques avancées
- Algorithme d'apprentissage adaptatif
- Thèmes et personnalisation avancée
- Types de questions autres que standard et QCM
- Optimisations de performance

## Architecture du prototype

Le prototype suivra une version simplifiée de l'architecture MVVM et Clean Architecture définie dans le document d'architecture, avec un focus sur les composants essentiels :

```
┌─────────────────────┐
│     UI (Compose)    │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│     ViewModels      │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│     Use Cases       │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│    Repositories     │
└──────────┬──────────┘
           │
┌──────────▼──────────┐    ┌──────────────────┐
│  Google Drive API   │◄───┤  Room Database   │
└─────────────────────┘    └──────────────────┘
```

## Composants techniques clés

### 1. Module d'authentification Google

```kotlin
// AuthRepository.kt
interface AuthRepository {
    suspend fun signIn(): Result<GoogleSignInAccount>
    suspend fun signOut()
    fun isSignedIn(): Boolean
    fun getAccount(): GoogleSignInAccount?
}

// AuthRepositoryImpl.kt
class AuthRepositoryImpl @Inject constructor(
    private val context: Context,
    private val googleSignInClient: GoogleSignInClient
) : AuthRepository {
    // Implémentation...
}

// SignInUseCase.kt
class SignInUseCase @Inject constructor(
    private val authRepository: AuthRepository
) {
    suspend operator fun invoke(): Result<GoogleSignInAccount> {
        return authRepository.signIn()
    }
}

// AuthViewModel.kt
class AuthViewModel @Inject constructor(
    private val signInUseCase: SignInUseCase,
    private val signOutUseCase: SignOutUseCase
) : ViewModel() {
    // État UI et logique...
}
```

### 2. Module d'accès à Google Drive

```kotlin
// DriveRepository.kt
interface DriveRepository {
    suspend fun listFiles(mimeType: String? = null): Result<List<DriveFile>>
    suspend fun getFileContent(fileId: String): Result<String>
    suspend fun searchFiles(query: String): Result<List<DriveFile>>
}

// DriveRepositoryImpl.kt
class DriveRepositoryImpl @Inject constructor(
    private val driveService: Drive,
    private val fileCache: FileCache
) : DriveRepository {
    // Implémentation...
}

// GetFlashcardFilesUseCase.kt
class GetFlashcardFilesUseCase @Inject constructor(
    private val driveRepository: DriveRepository
) {
    suspend operator fun invoke(): Result<List<DriveFile>> {
        return driveRepository.searchFiles("mimeType='text/yaml' OR mimeType='application/json'")
    }
}
```

### 3. Parseur YAML/JSON

```kotlin
// FlashcardParser.kt
interface FlashcardParser {
    fun parseYaml(content: String): Result<FlashcardDeck>
    fun parseJson(content: String): Result<FlashcardDeck>
}

// FlashcardParserImpl.kt
class FlashcardParserImpl @Inject constructor() : FlashcardParser {
    override fun parseYaml(content: String): Result<FlashcardDeck> {
        return try {
            val yaml = Yaml()
            val map = yaml.load<Map<String, Any>>(content)
            // Conversion en FlashcardDeck...
            Result.success(deck)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    override fun parseJson(content: String): Result<FlashcardDeck> {
        return try {
            val gson = Gson()
            val deck = gson.fromJson(content, FlashcardDeck::class.java)
            Result.success(deck)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 4. Base de données locale

```kotlin
// FlashcardDatabase.kt
@Database(
    entities = [FlashcardEntity::class, DeckEntity::class, CategoryEntity::class],
    version = 1
)
abstract class FlashcardDatabase : RoomDatabase() {
    abstract fun flashcardDao(): FlashcardDao
    abstract fun deckDao(): DeckDao
    abstract fun categoryDao(): CategoryDao
}

// FlashcardDao.kt
@Dao
interface FlashcardDao {
    @Query("SELECT * FROM flashcards WHERE deckId = :deckId")
    fun getFlashcardsByDeck(deckId: String): Flow<List<FlashcardEntity>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertFlashcards(flashcards: List<FlashcardEntity>)
    
    // Autres méthodes...
}
```

### 5. Interface utilisateur basique

```kotlin
// MainActivity.kt
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            CertForgeTheme {
                NavHost(
                    navController = rememberNavController(),
                    startDestination = "login"
                ) {
                    composable("login") { LoginScreen() }
                    composable("home") { HomeScreen() }
                    composable("deck/{deckId}") { backStackEntry ->
                        val deckId = backStackEntry.arguments?.getString("deckId") ?: ""
                        DeckDetailScreen(deckId = deckId)
                    }
                    composable("quiz/{deckId}") { backStackEntry ->
                        val deckId = backStackEntry.arguments?.getString("deckId") ?: ""
                        QuizScreen(deckId = deckId)
                    }
                }
            }
        }
    }
}

// LoginScreen.kt
@Composable
fun LoginScreen(
    viewModel: AuthViewModel = hiltViewModel()
) {
    val state by viewModel.state.collectAsState()
    
    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("CertForge", style = MaterialTheme.typography.h4)
        Spacer(modifier = Modifier.height(32.dp))
        Button(
            onClick = { viewModel.signIn() },
            enabled = !state.isLoading
        ) {
            Text("Se connecter avec Google")
        }
        if (state.isLoading) {
            Spacer(modifier = Modifier.height(16.dp))
            CircularProgressIndicator()
        }
        if (state.error != null) {
            Spacer(modifier = Modifier.height(16.dp))
            Text(
                text = state.error!!,
                color = MaterialTheme.colors.error
            )
        }
    }
}
```

### 6. Écran de quiz basique

```kotlin
// QuizScreen.kt
@Composable
fun QuizScreen(
    deckId: String,
    viewModel: QuizViewModel = hiltViewModel()
) {
    val state by viewModel.state.collectAsState()
    
    LaunchedEffect(deckId) {
        viewModel.loadQuiz(deckId)
    }
    
    Column(modifier = Modifier.fillMaxSize()) {
        // Barre de progression
        LinearProgressIndicator(
            progress = state.progress,
            modifier = Modifier.fillMaxWidth()
        )
        
        // Contenu principal
        Box(
            modifier = Modifier
                .weight(1f)
                .fillMaxWidth()
                .padding(16.dp)
        ) {
            when {
                state.isLoading -> {
                    CircularProgressIndicator(
                        modifier = Modifier.align(Alignment.Center)
                    )
                }
                state.currentFlashcard == null -> {
                    Text(
                        text = "Aucune flashcard disponible",
                        modifier = Modifier.align(Alignment.Center)
                    )
                }
                else -> {
                    val flashcard = state.currentFlashcard!!
                    when (flashcard.type) {
                        FlashcardType.STANDARD -> StandardFlashcard(
                            flashcard = flashcard,
                            onCorrect = { viewModel.markCorrect() },
                            onIncorrect = { viewModel.markIncorrect() }
                        )
                        FlashcardType.MCQ -> McqFlashcard(
                            flashcard = flashcard as McqFlashcardUiModel,
                            onSubmit = { selectedOptions ->
                                viewModel.submitMcqAnswer(selectedOptions)
                            }
                        )
                    }
                }
            }
        }
        
        // Actions
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            horizontalArrangement = Arrangement.SpaceBetween
        ) {
            Button(
                onClick = { viewModel.previousFlashcard() },
                enabled = state.canGoBack
            ) {
                Text("Précédent")
            }
            Button(
                onClick = { viewModel.nextFlashcard() },
                enabled = state.canGoNext
            ) {
                Text("Suivant")
            }
        }
    }
}
```

## Flux utilisateur du prototype

1. **Écran de connexion**
   - L'utilisateur se connecte avec son compte Google
   - L'application demande les permissions nécessaires pour Google Drive

2. **Écran d'accueil**
   - Liste des fichiers YAML/JSON trouvés dans Google Drive
   - Option pour actualiser la liste
   - Navigation vers les détails d'un deck

3. **Écran de détail du deck**
   - Affichage des métadonnées du deck
   - Liste des flashcards contenues dans le deck
   - Bouton pour démarrer une session de quiz

4. **Écran de quiz**
   - Affichage des flashcards une par une
   - Support des types standard et QCM
   - Boutons pour indiquer si la réponse est correcte/incorrecte
   - Navigation entre les flashcards

## Implémentation du prototype

### Étapes de développement

1. **Configuration du projet**
   - Création du projet Android avec Kotlin et Jetpack Compose
   - Configuration des dépendances essentielles
   - Mise en place de l'architecture de base

2. **Authentification Google**
   - Configuration du projet dans Google Cloud Console
   - Implémentation de l'authentification OAuth 2.0
   - Création de l'écran de connexion

3. **Intégration Google Drive**
   - Configuration de l'API Google Drive
   - Implémentation de la lecture des fichiers
   - Mise en place du cache local basique

4. **Parseur YAML/JSON**
   - Intégration des bibliothèques de parsing
   - Implémentation du parseur pour les formats standard et QCM
   - Gestion des erreurs de parsing

5. **Interface utilisateur**
   - Création des écrans principaux
   - Implémentation de la navigation
   - Développement des composants de flashcard et QCM

6. **Moteur de quiz**
   - Implémentation de la logique de session
   - Gestion des réponses utilisateur
   - Suivi basique de la progression

### Dépendances clés

```kotlin
dependencies {
    // Kotlin
    implementation("org.jetbrains.kotlin:kotlin-stdlib:1.6.10")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.0")
    
    // AndroidX
    implementation("androidx.core:core-ktx:1.7.0")
    implementation("androidx.appcompat:appcompat:1.4.1")
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.1")
    
    // Compose
    implementation("androidx.compose.ui:ui:1.2.0-alpha08")
    implementation("androidx.compose.material:material:1.2.0-alpha08")
    implementation("androidx.compose.ui:ui-tooling-preview:1.2.0-alpha08")
    implementation("androidx.navigation:navigation-compose:2.5.0-alpha03")
    
    // Hilt
    implementation("com.google.dagger:hilt-android:2.41")
    kapt("com.google.dagger:hilt-compiler:2.41")
    implementation("androidx.hilt:hilt-navigation-compose:1.0.0")
    
    // Room
    implementation("androidx.room:room-runtime:2.4.2")
    implementation("androidx.room:room-ktx:2.4.2")
    kapt("androidx.room:room-compiler:2.4.2")
    
    // Google
    implementation("com.google.android.gms:play-services-auth:20.1.0")
    implementation("com.google.api-client:google-api-client-android:1.33.0")
    implementation("com.google.apis:google-api-services-drive:v3-rev20220214-1.32.1")
    
    // Parsing
    implementation("org.yaml:snakeyaml:1.30")
    implementation("com.google.code.gson:gson:2.9.0")
    
    // Tests
    testImplementation("junit:junit:4.13.2")
    testImplementation("io.mockk:mockk:1.12.3")
    androidTestImplementation("androidx.test.ext:junit:1.1.3")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.4.0")
    androidTestImplementation("androidx.compose.ui:ui-test-junit4:1.2.0-alpha08")
}
```

## Tests du prototype

### Tests manuels

1. **Test d'authentification**
   - Vérifier que l'utilisateur peut se connecter avec Google
   - Vérifier que les tokens sont correctement stockés
   - Vérifier que la déconnexion fonctionne

2. **Test d'accès à Google Drive**
   - Vérifier que l'application peut lister les fichiers
   - Vérifier que l'application peut lire le contenu des fichiers
   - Vérifier le comportement en cas d'erreur réseau

3. **Test du parseur**
   - Vérifier le parsing des fichiers YAML valides
   - Vérifier le parsing des fichiers JSON valides
   - Vérifier la gestion des erreurs de format

4. **Test de l'interface utilisateur**
   - Vérifier la navigation entre les écrans
   - Vérifier l'affichage des flashcards standard
   - Vérifier l'affichage et l'interaction avec les QCM

5. **Test du quiz**
   - Vérifier le déroulement d'une session de quiz
   - Vérifier la gestion des réponses correctes/incorrectes
   - Vérifier la navigation entre les flashcards

### Scénarios de test

#### Scénario 1 : Première utilisation
1. L'utilisateur lance l'application
2. L'utilisateur se connecte avec Google
3. L'application demande les permissions pour Google Drive
4. L'application affiche les fichiers YAML/JSON disponibles
5. L'utilisateur sélectionne un fichier
6. L'application affiche les détails du deck
7. L'utilisateur démarre une session de quiz
8. L'utilisateur répond à quelques flashcards
9. L'utilisateur termine la session

#### Scénario 2 : Utilisation hors ligne
1. L'utilisateur se connecte à l'application
2. L'application charge les fichiers depuis Google Drive
3. L'utilisateur active le mode avion
4. L'utilisateur peut toujours accéder aux decks précédemment chargés
5. L'utilisateur peut effectuer une session de quiz

## Livrables du prototype

1. **Code source** du prototype fonctionnel
2. **APK** installable sur appareil Android
3. **Documentation** expliquant les fonctionnalités et limitations
4. **Guide d'utilisation** pour tester le prototype
5. **Rapport de test** documentant les résultats des tests manuels

## Critères de validation du prototype

Le prototype sera considéré comme validé si :

1. L'authentification Google fonctionne correctement
2. L'application peut lire et parser des fichiers YAML/JSON depuis Google Drive
3. L'interface utilisateur permet de naviguer entre les écrans principaux
4. Les flashcards standard et QCM s'affichent correctement
5. Une session de quiz basique peut être complétée
6. L'application fonctionne en mode hors ligne avec les données précédemment chargées

## Prochaines étapes après validation

1. Recueillir les retours utilisateurs sur le prototype
2. Ajuster l'architecture et les fonctionnalités si nécessaire
3. Développer les fonctionnalités complètes selon le plan de développement
4. Améliorer l'interface utilisateur et l'expérience globale
5. Implémenter les fonctionnalités avancées (statistiques, synchronisation bidirectionnelle, etc.)
