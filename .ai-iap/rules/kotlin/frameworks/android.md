# Android Development with Kotlin

> **Scope**: Android apps using Kotlin (Google's recommended language since 2019)
> **Applies to**: Kotlin files in Android projects
> **Extends**: kotlin/architecture.md, kotlin/code-style.md

## CRITICAL REQUIREMENTS (AI: Verify ALL before generating code)

> **ALWAYS**: Use ViewBinding (NOT findViewById or Kotlin synthetics)
> **ALWAYS**: Use `by viewModels()` delegation (NOT manual ViewModelProvider)
> **ALWAYS**: Clean up Fragment bindings in `onDestroyView` (memory leak prevention)
> **ALWAYS**: Use `lifecycleScope` for coroutines (auto-cancellation)
> **ALWAYS**: Use StateFlow or LiveData for reactive UI
> 
> **NEVER**: Use `lateinit` for ViewBinding in Fragments (causes memory leaks)
> **NEVER**: Use manual ViewModel creation (use delegation)
> **NEVER**: Use `findViewById()` (use ViewBinding)
> **NEVER**: Use `GlobalScope` (not lifecycle-aware)
> **NEVER**: Block main thread with synchronous operations

## Pattern Selection

| Pattern | Use When | Keywords |
|---------|----------|----------|
| **StateFlow** | New apps, coroutine support | Modern, Kotlin-first, cold streams |
| **LiveData** | Java interop, existing codebase | Lifecycle-aware out of the box |
| **Activity lateinit** | ViewBinding in Activities | Safe (onCreate before access) |
| **Fragment nullable** | ViewBinding in Fragments | Memory leak prevention |

## Core Patterns

### Activities

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding  // lateinit: safe in Activities
    private val viewModel: MainViewModel by viewModels()  // Lazy: survives rotation
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        viewModel.state.observe(this) { state ->  // 'this' = LifecycleOwner
            when (state) {
                is UiState.Loading -> showLoading()
                is UiState.Success -> showData(state.data)
                is UiState.Error -> showError(state.message)
            }
        }
    }
}
```

### Fragments (CRITICAL: Memory Leak Prevention)

```kotlin
class UserFragment : Fragment(R.layout.fragment_user) {
    private var _binding: FragmentUserBinding? = null  // Nullable: view destroyed before fragment
    private val binding get() = _binding!!  // Safe: only between onCreateView-onDestroyView
    private val viewModel: UserViewModel by viewModels()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        _binding = FragmentUserBinding.bind(view)
        setupUI()
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null  // CRITICAL: Release view reference
    }
}
```

### ViewModels

```kotlin
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _state = MutableStateFlow<UiState<User>>(UiState.Loading)
    val state: StateFlow<UiState<User>> = _state.asStateFlow()  // Read-only for UI
    
    fun loadUser() {
        viewModelScope.launch {  // Cancelled when ViewModel cleared
            _state.value = UiState.Loading
            repository.getUser()
                .onSuccess { _state.value = UiState.Success(it) }
                .onFailure { _state.value = UiState.Error(it.message ?: "") }
        }
    }
}

sealed class UiState<out T> {
    data object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val message: String) : UiState<Nothing>()
}
```

## Jetpack Compose

```kotlin
@Composable
fun UserScreen(viewModel: UserViewModel = viewModel()) {
    val state by viewModel.state.collectAsStateWithLifecycle()
    
    when (state) {
        is UiState.Loading -> LoadingIndicator()
        is UiState.Success -> UserDetails((state as UiState.Success).data)
        is UiState.Error -> ErrorView((state as UiState.Error).message)
    }
}
```

## Room Database

```kotlin
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey val id: Long,
    @ColumnInfo(name = "name") val name: String
)

@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    suspend fun getAll(): List<UserEntity>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: UserEntity)
}
```

## Dependency Injection (Hilt)

```kotlin
@HiltAndroidApp
class MyApplication : Application()

@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext context: Context) =
        Room.databaseBuilder(context, AppDatabase::class.java, "db").build()
}

@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel()
```

## Common AI Mistakes (DO NOT MAKE THESE ERRORS)

| Mistake | ❌ Wrong | ✅ Correct | Why Critical |
|---------|---------|-----------|--------------|
| **Fragment Binding** | `lateinit var binding` | `var _binding: ..? = null` | Memory leak |
| **ViewModel Creation** | `ViewModelProvider(this)` | `by viewModels()` | Manual lifecycle management |
| **Coroutine Scope** | `GlobalScope.launch` | `lifecycleScope.launch` | Not lifecycle-aware |
| **findViewById** | `findViewById<TextView>` | ViewBinding | Type-unsafe, verbose |
| **Leaked Observers** | No lifecycle owner | `.observe(viewLifecycleOwner)` | Crash after destroy |

### Anti-Pattern: Fragment Memory Leak (CRITICAL)

```kotlin
// ❌ WRONG: Memory leak
class UserFragment : Fragment() {
    private lateinit var binding: FragmentUserBinding  // Fragment kept in memory!
}

// ✅ CORRECT: Proper cleanup
class UserFragment : Fragment() {
    private var _binding: FragmentUserBinding? = null
    private val binding get() = _binding!!
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null  // Release view reference
    }
}
```

### Anti-Pattern: GlobalScope (LIFECYCLE DISASTER)

```kotlin
// ❌ WRONG: Never cancelled
GlobalScope.launch {
    val data = fetchData()
    updateUI(data)  // Crash if Activity destroyed
}

// ✅ CORRECT: Lifecycle-aware
lifecycleScope.launch {
    val data = fetchData()
    updateUI(data)  // Cancelled if Activity destroyed
}
```

## AI Self-Check (Verify BEFORE generating Android code)

- [ ] Using ViewBinding (NOT findViewById)?
- [ ] Nullable binding pattern in Fragments?
- [ ] Cleaning up binding in onDestroyView?
- [ ] Using `by viewModels()` delegation?
- [ ] StateFlow or LiveData for state?
- [ ] lifecycleScope for coroutines?
- [ ] Sealed classes for UI state?
- [ ] Hilt for dependency injection?
- [ ] Room for local database?
- [ ] No memory leaks in Fragments?

## Key Libraries

| Library | Purpose | Keywords |
|---------|---------|----------|
| ViewBinding | View access | Type-safe, null-safe |
| Lifecycle | Lifecycle awareness | `lifecycleScope`, `viewLifecycleOwner` |
| ViewModel | Survive rotation | `by viewModels()` |
| StateFlow/LiveData | Reactive UI | Observable state |
| Room | Local database | `@Entity`, `@Dao`, `@Database` |
| Hilt | Dependency injection | `@Inject`, `@HiltViewModel` |
| Compose | Declarative UI | `@Composable`, `collectAsStateWithLifecycle` |
| Navigation | Multi-screen | `NavController`, `navigate()` |

## Best Practices

**MUST**:
- ViewBinding in all UI components
- Nullable binding in Fragments + cleanup
- `by viewModels()` delegation
- lifecycleScope for coroutines
- StateFlow/LiveData for reactive UI

**SHOULD**:
- Hilt for DI
- Room for local storage
- Jetpack Navigation
- Sealed classes for state
- Data classes for models

**AVOID**:
- `lateinit` for Fragment ViewBinding
- Manual ViewModel creation
- `findViewById()`
- `GlobalScope`
- Main thread blocking
