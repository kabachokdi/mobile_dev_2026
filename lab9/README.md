<div align="center">

**МИНИСТЕРСТВО НАУКИ И ВЫСШЕГО ОБРАЗОВАНИЯ РОССИЙСКОЙ ФЕДЕРАЦИИ**  
**ФЕДЕРАЛЬНОЕ ГОСУДАРСТВЕННОЕ БЮДЖЕТНОЕ ОБРАЗОВАТЕЛЬНОЕ УЧРЕЖДЕНИЕ ВЫСШЕГО ОБРАЗОВАНИЯ**  
**«САХАЛИНСКИЙ ГОСУДАРСТВЕННЫЙ УНИВЕРСИТЕТ»**

<br>
<br>

Институт естественных наук и техносферной безопасности  
Кафедра информатики  
Бычков Дмитрий Николаевич

<br>
<br>
<br>
<br>

Лабораторная работа №8
Перенос логики списка задач из Activity в ViewModel. Использование StateFlow для хранения состояния
01.03.02 Прикладная математика и информатика  
3 Курс

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

<div align="right">
Научный руководитель
<br>
Соболев Евгений Игоревич
</div>

<br>
<br>
<br>

г. Южно-Сахалинск  
2026 г.

</div>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
Листинг Color.kt
<br>

```kotlin
package com.example.lab9.ui.theme
import androidx.compose.ui.graphics.Color
import androidx.compose.material3.*

// Светлая тема
val LightColors = lightColorScheme(
    primary = Color(0xFF05DEFF),
    onPrimary = Color(0xFFFFFFFF),
    primaryContainer = Color(0xFF89F8C7),
    onPrimaryContainer = Color(0xFF002114),
    secondary = Color(0xFFFF2605),
    onSecondary = Color(0xFFFFFFFF),
    secondaryContainer = Color(0xFFCFE9DD),
    onSecondaryContainer = Color(0xFF0A1F19),
    tertiary = Color(0xFF3A637A),
    onTertiary = Color(0xFFFFFFFF),
    tertiaryContainer = Color(0xFFC1E8FF),
    onTertiaryContainer = Color(0xFF001E2C),
    background = Color(0xFFF4FBF5),
    onBackground = Color(0xFF161D1A),
    surface = Color(0xFFF4FBF5),
    onSurface = Color(0xFF161D1A)
)

// Тёмная тема
val DarkColors = darkColorScheme(
    primary = Color(0xFF6CDBB0),
    onPrimary = Color(0xFF003825),
    primaryContainer = Color(0xFF005239),
    onPrimaryContainer = Color(0xFF89F8C7),
    secondary = Color(0xFFB3CCC1),
    onSecondary = Color(0xFF1F352D),
    secondaryContainer = Color(0xFF354B43),
    onSecondaryContainer = Color(0xFFCFE9DD),
    tertiary = Color(0xFF9DC9E5),
    onTertiary = Color(0xFF003549),
    tertiaryContainer = Color(0xFF1F4B63),
    onTertiaryContainer = Color(0xFFC1E8FF),
    background = Color(0xFF161D1A),
    onBackground = Color(0xFFE1E3DF),
    surface = Color(0xFF161D1A),
    onSurface = Color(0xFFE1E3DF)
)

```

<br>

Листинг Theme.kt

<br>

```kotlin


import android.app.Activity
import android.os.Build
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.graphics.toArgb
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.platform.LocalView
import androidx.core.view.WindowCompat
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun ThemeSwitcherTheme(
    viewModel: ThemeViewModel = viewModel(),
    content: @Composable () -> Unit
) {
    val context = LocalContext.current
    val themeMode by viewModel.themeMode.collectAsState()

    // Определяем, тёмная ли тема сейчас
    val darkTheme = when (themeMode) {
        ThemeMode.SYSTEM -> isSystemInDarkTheme()
        ThemeMode.LIGHT -> false
        ThemeMode.DARK -> true
    }

    val colorScheme = when {
        Build.VERSION.SDK_INT >= Build.VERSION_CODES.S && themeMode== ThemeMode.SYSTEM -> {
            if (darkTheme ) dynamicDarkColorScheme(context)
            else  dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColors
        else -> LightColors
    }

    val view = LocalView.current
    if (!view.isInEditMode) {
        SideEffect {
            val window = (view.context as Activity).window
            window.statusBarColor = colorScheme.primary.toArgb()
            WindowCompat.getInsetsController(window, view)
                .isAppearanceLightStatusBars = !darkTheme
        }
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography(),
        content = content
    )
}
```

<br>
Листинг SettingsManager.kt

<br>

```kotlin



import android.content.Context
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore
import com.example.lab9.ui.theme.ThemeMode
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

val Context.dataStore by preferencesDataStore(name = "settings")

class SettingsManager(private val context: Context) {

    companion object {
        val THEME_MODE_KEY = stringPreferencesKey("theme_mode")
        private const val DEFAULT_MODE = "SYSTEM"
    }

    val themeMode: Flow<ThemeMode> = context.dataStore.data
        .map { preferences ->
            val modeName = preferences[THEME_MODE_KEY] ?: DEFAULT_MODE
            try {
                ThemeMode.valueOf(modeName)
            } catch (e: IllegalArgumentException) {
                ThemeMode.SYSTEM
            }
        }

    suspend fun saveThemeMode(mode: ThemeMode) {
        context.dataStore.edit { preferences ->
            preferences[THEME_MODE_KEY] = mode.name
        }
    }
}

```

<br>

Листинг ThemeViewModel.kt

<br>

```kotlin
class ThemeViewModel(
    private val settingsManager: SettingsManager
) : ViewModel() {

    val themeMode: StateFlow<ThemeMode> = settingsManager.themeMode
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = ThemeMode.SYSTEM
        )

    fun setThemeMode(mode: ThemeMode) {
        viewModelScope.launch {
            settingsManager.saveThemeMode(mode)
        }
    }
}
```

<br>

Листинг main
<br>

```kotlin

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel
import com.example.lab9.ui.theme.ThemeMode
import com.example.lab9.ui.theme.ThemeSwitcherTheme
import com.example.lab9.ui.theme.ThemeViewModel


class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val settingsManager = SettingsManager(this)

        setContent {
            ThemeSwitcherTheme(
                viewModel = viewModel(factory = ThemeViewModelFactory(settingsManager))
            ) {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    AppNavigation()
                }
            }
        }
    }
}

@Composable
fun AppNavigation(viewModel: ThemeViewModel = viewModel()) {
    var showSettings by remember { mutableStateOf(false) }

    if (showSettings) {
        SettingsScreen(
            onBack = { showSettings = false },
            viewModel = viewModel
        )
    } else {
        ThemeScreen(
            onOpenSettings = { showSettings = true },
            viewModel = viewModel
        )
    }
}

@Composable
fun ThemeScreen(
    onOpenSettings: () -> Unit,
    viewModel: ThemeViewModel = viewModel()
) {
    val themeMode by viewModel.themeMode.collectAsState()

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = "Текущая тема: ${
                when (themeMode) {
                    ThemeMode.SYSTEM -> "Системная"
                    ThemeMode.LIGHT -> "Светлая"
                    ThemeMode.DARK -> "Тёмная"
                }
            }",
            style = MaterialTheme.typography.headlineMedium
        )

        Spacer(modifier = Modifier.height(24.dp))

        Button(onClick = onOpenSettings) {
            Text("Открыть настройки темы")
        }

        Spacer(modifier = Modifier.height(32.dp))

        Card(
            modifier = Modifier.fillMaxWidth(),
            elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
        ) {
            Column(modifier = Modifier.padding(16.dp)) {
                Text(
                    text = "Пример карточки",
                    style = MaterialTheme.typography.titleLarge
                )
                Text(
                    text = "Это демонстрация того, как тема влияет на цвета компонентов.",
                    style = MaterialTheme.typography.bodyMedium
                )
            }
        }

        Spacer(modifier = Modifier.height(24.dp))

        Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
            Button(
                onClick = {},
                colors = ButtonDefaults.buttonColors(
                    containerColor = MaterialTheme.colorScheme.secondary
                )
            ) {
                Text("Кнопка 1")
            }
            OutlinedButton(onClick = {}) {
                Text("Кнопка 2")
            }
        }
    }
}

@Composable
fun SettingsScreen(
    onBack: () -> Unit,
    viewModel: ThemeViewModel = viewModel()
) {
    val themeMode by viewModel.themeMode.collectAsState()

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        Text(
            text = "Настройки темы",
            style = MaterialTheme.typography.headlineLarge
        )

        Spacer(modifier = Modifier.height(24.dp))

        val options = listOf(
            ThemeMode.SYSTEM to "Системная",
            ThemeMode.LIGHT to "Светлая",
            ThemeMode.DARK to "Тёмная"
        )

        options.forEach { (mode, label) ->
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(vertical = 8.dp),
                verticalAlignment = Alignment.CenterVertically
            ) {
                RadioButton(
                    selected = themeMode == mode,
                    onClick = { viewModel.setThemeMode(mode) }
                )
                Spacer(modifier = Modifier.width(8.dp))
                Text(
                    text = label,
                    style = MaterialTheme.typography.bodyLarge,
                    modifier = Modifier.clickable { viewModel.setThemeMode(mode) }
                )
            }
        }

        Spacer(modifier = Modifier.height(32.dp))

        Button(onClick = onBack) {
            Text("Назад")
        }
    }
}
```

<br>

<img width="448" height="845" alt="image" src="https://github.com/user-attachments/assets/8dddd83f-2bb2-4203-90f2-3ccaa556ebfc" />
<img width="460" height="871" alt="image" src="https://github.com/user-attachments/assets/41165358-bb08-4b0b-8c42-3b0ae5dc3292" />
<img width="442" height="834" alt="image" src="https://github.com/user-attachments/assets/94d550e7-c310-4470-975e-99063ea349f7" />

Контрольные вопросы:
<br>
## 1  Как в Compose определить, какая тема активна в данный момент (тёмная/светлая)?
В Jetpack Compose текущая тема определяется через MaterialTheme.colorScheme и доступ к системному флагу через isSystemInDarkTheme().
## 2  Что такое MaterialTheme.colorScheme и какие основные цвета он содержит?
MaterialTheme.colorScheme – это объект типа ColorScheme, предоставляющий все основные цвета, используемые в Material Design 3 для стилизации компонентов. Он централизует цветовое оформление приложения.

Основные группы цветов, содержащиеся в ColorScheme:

- **primary** – основной, самый заметный цвет (кнопки, заголовки, активные элементы).

- **onPrimary** – цвет текста/иконок на primary-фоне.

- **primaryContainer** – более светлый (в светлой теме) или более тёмный (в тёмной) вариант primary, используется для контейнеров, выделяющихся, но не ярких.

- **onPrimaryContainer** – цвет текста на primaryContainer.

- **secondary** – второстепенный цвет (например, фильтры, чипы).

- **onSecondary** – цвет текста на secondary.

- **secondaryContainer** – контейнерный вариант secondary.

- **onSecondaryContainer** – текст на secondaryContainer.

- **tertiary** – третий акцентный цвет (используется реже, для разнообразия).

- **onTertiary** – текст на tertiary.

- **tertiaryContainer** – контейнерный tertiary.

- **onTertiaryContainer** – текст на нём.

- **background** – основной фон приложения.

- **onBackground** – цвет текста/элементов на фоне.

- **surface** – цвет поверхностей (карточек, диалогов).

- **onSurface** – текст на surface.

- **surfaceVariant** – альтернативная поверхность.

- **onSurfaceVariant** – текст на ней.

- **error** – цвет ошибок.

- **onError** – текст/иконки на error.

- **outline** – цвет обводки (например, у OutlinedTextField).

- **outlineVariant** – более мягкая обводка.

- **inverseSurface / inverseOnSurface / inversePrimary** – инверсные цвета для контраста (например, в Snackbar).

Эти цвета автоматически настраиваются с учётом светлой/тёмной темы и могут быть динамическими (на Android 12+).

## 3 Как сохранить выбор темы пользователя между сессиями работы приложения?
Сохранение пользовательских настроек основано на принципе постоянного хранилища ключ-значение. Это решается двумя способами:

SharedPreferences – синхронное/асинхронное хранение простых пар «ключ‑значение» в XML-файле.

DataStore Preferences – современный асинхронный API, построенный на Kotlin Coroutines и Flow. 

## 4 В чём разница между isSystemInDarkTheme() и сохранённым пользовательским выбором?

isSystemInDarkTheme() – это функция Compose, которая возвращает текущую системную настройку Android: включён ли тёмный режим на устройстве. Она не зависит от вашего приложения и может меняться пользователем в любое время в настройках телефона.

Сохранённый пользовательский выбор – это значение, которое ваше приложение хранит в DataStore/SharedPreferences. Оно отражает, какую тему выбрал пользователь внутри самого приложения (например, «всегда тёмная», «всегда светлая» или «как в системе»).

## 5 Что такое динамические цвета (dynamic color) и на каких версиях Android они доступны?
Динамические цвета (Material You) – это возможность генерировать цветовую схему на основе цветов текущих обоев устройства или заранее заданного цвета-источника. Вместо статической палитры приложение автоматически получает гармонирующие с обоями оттенки для primary, secondary, tertiary и их производных.

В Compose для этого используются функции:

dynamicLightColorScheme(context) – для светлой темы.

dynamicDarkColorScheme(context) – для тёмной темы.

Они возвращают ColorScheme, основанный на динамических цветах.

Доступность: динамические цвета поддерживаются на устройствах под управлением Android 12 (API 31) и выше, где присутствует библиотека Material You.


<br>
Вывод: Было освоено переключение тем, работа с постоянным хранилищеим DataStore Preferences 
