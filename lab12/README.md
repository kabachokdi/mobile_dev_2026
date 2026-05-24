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

Лабораторная работа №12
Выполнение длительных операций (симуляция загрузки) с использованием viewModelScope
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

Листинг Uistate

<br>

```kotlin
package com.example.myapplication.ui
import com.example.todoapp.database.TaskEntity

sealed class TasksUiState {
    object Loading : TasksUiState()
    data class Success(val tasks: List<TaskEntity>) : TasksUiState()
    data class Error(val message: String) : TasksUiState()
}
```

<br>
Листинг mainViewModel
<br>

```kotlin
package com.example.myapplication

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.myapplication.data.TaskRepository
import com.example.myapplication.ui.TasksUiState
import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.MutableSharedFlow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asSharedFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class MainViewModel (
    private val repository: TaskRepository
): ViewModel() {

    private val _errorEvent = MutableSharedFlow<String>()
    val errorEvent = _errorEvent.asSharedFlow()
    private val _uiState = MutableStateFlow<TasksUiState>(TasksUiState.Loading)
    val uiState: StateFlow<TasksUiState> = _uiState.asStateFlow()

    var needed = MutableStateFlow<List<TaskEntity>>(emptyList())
    init{
        loadTasks()
    }
    fun loadTasks(){
        viewModelScope.launch {
            _uiState.value = TasksUiState.Loading
            try {
                delay(4000) // 4 секунды
                val tasks = repository.getTasksOnce()
                _uiState.value = TasksUiState.Success(tasks)

            } catch (e: Exception) {
                _uiState.value = TasksUiState.Error(e.message ?: "Ошибка загрузки")
            }
        }
    }


    val tasks: StateFlow<List<TaskEntity>> = repository.getAllTasks().stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(3000),

        initialValue = emptyList()
    )
    fun loadTasksForCategory(category: String) {
        viewModelScope.launch{
            val par = repository.loadTaskByHeader(category)
        }
    }

    fun addTask(title: TaskEntity) {
        viewModelScope.launch {
            repository.addTask(title)
                .onFailure { e ->
                    _errorEvent.emit("Ошибка добавления: ${e.localizedMessage}")
                }
        }
    }
    fun deleteTask(task: TaskEntity) {
        viewModelScope.launch {
            repository.deleteTask(task)
                .onFailure { e ->
                    _errorEvent.emit("Ошибка удаления: ${e.localizedMessage}")
                }
        }
    }

    fun updateTask(task: TaskEntity) {
        viewModelScope.launch {
            repository.updateTask(task).onFailure { e -> _errorEvent.emit( "Ошибка обновления: ${e.localizedMessage}") }
        }
    }

    fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        viewModelScope.launch {
            repository.toggleTaskCompletion(task, isCompleted)
                .onFailure { e ->
                    _errorEvent.emit("Ошибка обновления: ${e.localizedMessage}")
                }
        }
    }

    fun deleteAllTasks() {
        viewModelScope.launch {
            repository.deleteAllTasks()
                .onFailure { e ->
                    _errorEvent.emit("Ошибка удаления всех: ${e.localizedMessage}")
                }
        }
    }

    fun refresh() {
        loadTasks()
    }
    }
```

<br>
Листинг taskReposytory

```kotlin
interface TaskRepository {
    fun getAllTasks(): Flow<List<TaskEntity>>
    suspend fun addTask(task: TaskEntity): Result<Unit>

    suspend fun deleteTask(task: TaskEntity): Result<Unit>
    suspend fun updateTask(task: TaskEntity): Result<Unit>
    suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean): Result<Unit>
    suspend fun deleteAllTasks(): Result<Unit>
    suspend fun loadTaskByHeader(header: String): List<TaskEntity>
    suspend fun getTasksOnce(): List<TaskEntity>
}
```

<br>
Листинг taskReposytoryIMPL
<br>

```kotlin

class TaskRepositoryImpl(
    private val taskDao: TaskDao
) : TaskRepository {

    override fun getAllTasks(): Flow<List<TaskEntity>> = taskDao.getAllTasks()


    override suspend fun addTask(task: TaskEntity): Result<Unit> {
        return try {
            val task = TaskEntity(id = task.id,title = task.title, header =task.header)
            taskDao.insertTask(task)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }

    override suspend fun deleteTask(task: TaskEntity): Result<Unit> {
        return try {
            taskDao.deleteTask(task)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }


    override suspend fun updateTask(task: TaskEntity): Result<Unit> {
        return try {
            taskDao.updateTask(task)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }

    override suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean): Result<Unit> {
        return try {
            val updatedTask = task.copy(isCompleted = isCompleted)
            taskDao.updateTask(updatedTask)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }


    override suspend fun deleteAllTasks(): Result<Unit> {
        return try {
            taskDao.deleteAll()
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }

    override suspend fun loadTaskByHeader(header: String): List<TaskEntity>{

        val lst = taskDao.byheader(header)
        return lst
    }

    override suspend fun getTasksOnce(): List<TaskEntity> {
        return taskDao.getAllTasks().first() // first() приостановится до первого элемента
    }



}

```

<br>
Листинг activity_main.xml
<br>

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">
    <!-- Поле ввода и кнопка добавления (как в Лаб.5) -->
    <EditText
        android:id="@+id/editTextheader"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="8dp"
        android:hint="Введите заголовок" />

    <EditText
        android:id="@+id/editTextTask"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="8dp"
        android:hint="Введите задачу" />


    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="70dp"
        android:orientation="horizontal">

        <Button
            android:id="@+id/buttonAddTask"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            android:layout_weight="1"
            android:text="Добавить задачу" />

        <Button
            android:id="@+id/buttonsearch"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            android:layout_weight="1"
            android:text="Поиск" />

        <Button
            android:id="@+id/buttonrefresh"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="Обновить" />

    </LinearLayout>
    <EditText
        android:id="@+id/editTextSearch"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="8dp"
        android:hint="Введите" />

    <!-- RecyclerView для списка задач -->
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerViewTasks"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:visibility="gone"/>
        <!-- Shimmer-контейнер для скелетонов -->
        <com.facebook.shimmer.ShimmerFrameLayout
            android:id="@+id/shimmerLayout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:visibility="gone">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical">
                <include layout="@layout/skeleton_item_task" />
                <include layout="@layout/skeleton_item_task" />
                <include layout="@layout/skeleton_item_task" />
                <include layout="@layout/skeleton_item_task" />
                <include layout="@layout/skeleton_item_task" />
                <include layout="@layout/skeleton_item_task" />

            </LinearLayout>
        </com.facebook.shimmer.ShimmerFrameLayout>

        <TextView
            android:id="@+id/textError"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:text="Ошибка загрузки"
            android:visibility="gone"/>

    </FrameLayout>

</LinearLayout>
```

<br>
Листинг main
<br>

```kotlin
package com.example.myapplication
import android.content.Intent
import android.os.Bundle
import android.view.View
import android.widget.Button
import android.widget.EditText
import android.widget.ProgressBar
import android.widget.TextView
import android.widget.Toast
import androidx.activity.result.ActivityResultLauncher
import androidx.activity.result.contract.ActivityResultContracts
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.myapplication.data.TaskRepository
import com.example.myapplication.data.TaskRepositoryImpl
import com.example.myapplication.database.AppDatabase
import com.example.myapplication.ui.TasksUiState
import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.flow.collectLatest
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private var tasks = emptyList<TaskEntity>()
    private lateinit var adapter: TaskAdapter
    private lateinit var  Detailslauncher: ActivityResultLauncher<Intent>
    var  ready = 0
    class MainViewModelFactory(
        private val repository: TaskRepository
    ) : ViewModelProvider.Factory {
        override fun <T : ViewModel> create(modelClass: Class<T>): T {
            if (modelClass.isAssignableFrom(MainViewModel::class.java)) {
                @Suppress("UNCHECKED_CAST")
                return MainViewModel(repository) as T
            }
            throw IllegalArgumentException("Unknown ViewModel class")
        }
    }

    private val database by lazy { AppDatabase.getInstance(this) }
    private val repository by lazy { TaskRepositoryImpl(database.taskDao()) }
    private val viewModel: MainViewModel by viewModels {
        MainViewModelFactory(repository)
    }


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val editTextheader = findViewById<EditText>(R.id.editTextheader)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)
        val recyclerView = findViewById<RecyclerView>(R.id.recyclerViewTasks)
        val searchbutton = findViewById<Button>(R.id.buttonsearch)
        val searchedd = findViewById<EditText>(R.id.editTextSearch)
        val mbutton = findViewById<Button>(R.id.buttonrefresh)
            // Создание лаунчера DetailActictivity, с возвратом разных result code
        Detailslauncher  = registerForActivityResult(ActivityResultContracts.StartActivityForResult()){
                result ->
            if (result.resultCode== RESULT_CANCELED){
                viewModel.deleteTask(result.data?.getParcelableExtra<TaskEntity>("Task") ?: viewModel.tasks.value[0])
                adapter.updateData(viewModel.tasks.value)
                adapter.notifyItemRemoved(result.data?.getIntExtra("pos", 0) ?: 0)
                Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
            }
            if (result.resultCode == 3){
               val newTask = result.data?.getParcelableExtra<TaskEntity>("Task") ?:viewModel.tasks.value[result.data?.getIntExtra("pos", 0) ?: 0]
                newTask.title = result.data?.getStringExtra("nw") ?: ""
                viewModel.updateTask(newTask)

                adapter.updateData(viewModel.tasks.value)
               Toast.makeText(this, "Задача изменена", Toast.LENGTH_SHORT).show()
            }
        }


        // Настройка RecyclerView
        recyclerView.layoutManager = LinearLayoutManager(this)

        adapter = TaskAdapter(tasks,

            { position,task ->
                viewModel.deleteTask(task)
                adapter.updateData(viewModel.tasks.value)
                adapter.notifyItemRemoved(position)
                Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
            },
            ///
            { chk,task, done->
                viewModel.toggleTaskCompletion(task,chk)
               },

            {position, Task ->
                val intent = Intent(this, DetailActivity::class.java)
                intent.putExtra("Task", Task)
                intent.putExtra("pos",position)
                Detailslauncher.launch(intent)

            }
        )
        recyclerView.adapter = adapter

            lifecycleScope.launch {
                repeatOnLifecycle(Lifecycle.State.STARTED) {
                    viewModel.tasks.collect { tasks ->
                        adapter.updateData(tasks)
                    }

                }
            }

        lifecycleScope.launch {
            viewModel.errorEvent.collectLatest { message ->
                Toast.makeText(this@MainActivity, message, Toast.LENGTH_SHORT).show()
            }
        }


        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    when (state) {
                        is TasksUiState.Loading -> {
                            findViewById<RecyclerView>(R.id.recyclerViewTasks).visibility = View.GONE
                            findViewById<com.facebook.shimmer.ShimmerFrameLayout>(R.id.shimmerLayout).apply {
                                visibility = View.VISIBLE
                                startShimmer()   // запуск анимации
                            }
                            findViewById<TextView>(R.id.textError).visibility = View.GONE
                        }
                        is TasksUiState.Success -> {
                            findViewById<com.facebook.shimmer.ShimmerFrameLayout>(R.id.shimmerLayout).apply {
                                stopShimmer()    // остановка анимации
                                visibility = View.GONE
                            }
                            findViewById<RecyclerView>(R.id.recyclerViewTasks).apply {
                                visibility = View.VISIBLE
                            }
                            findViewById<TextView>(R.id.textError).visibility = View.GONE
                            adapter.updateData(state.tasks)
                        }
                        is TasksUiState.Error -> {
                            findViewById<com.facebook.shimmer.ShimmerFrameLayout>(R.id.shimmerLayout).apply {
                                stopShimmer()
                                visibility = View.GONE
                            }
                            findViewById<RecyclerView>(R.id.recyclerViewTasks).visibility = View.GONE
                            findViewById<TextView>(R.id.textError).apply {
                                visibility = View.VISIBLE
                                text = state.message
                            }
                        }
                    }
                }
            }
        }

        // Добавление задачи
        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            val header = editTextheader.text.toString()
            if (task.isNotBlank()) {
                viewModel.addTask(TaskEntity(header=header, title =task))
                editTextTask.text.clear()
                editTextheader.text.clear()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }

        searchbutton.setOnClickListener {
            val header = searchedd.text.toString()
            if (header.isNotBlank()) {
                viewModel.loadTasksForCategory(header)
                tasks = viewModel.needed.value
                for ( t in tasks){
                    Toast.makeText(this, t.title, Toast.LENGTH_SHORT).show()
                }
                searchedd.text.clear()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }

        findViewById<Button>(R.id.buttonrefresh).setOnClickListener {
            viewModel.refresh()
        }

        }
    }
```

<br>
Загрузка


<img width="341" height="711" alt="image" src="https://github.com/user-attachments/assets/ceba2d18-f15c-40bb-b0b9-2b0f8227caf2" />

<img width="353" height="671" alt="image" src="https://github.com/user-attachments/assets/65d7be1d-d07e-4bbe-86cd-4bcac1890133" />

<img width="356" height="714" alt="image" src="https://github.com/user-attachments/assets/22a44911-5005-4512-b7c4-82e15a05780e" />

<br>

## 1 Почему длительные операции нельзя выполнять в главном потоке?
Главный поток отвечает за отрисовку UI и обработку событий. Долгая операция блокирует его, интерфейс зависает, система может показать диалог «Приложение не отвечает» (ANR).

<br>

## 2 Что такое viewModelScope и как он связан с жизненным циклом ViewModel?
viewModelScope — встроенный CoroutineScope, привязанный к ViewModel. Все корутины, запущенные в нём, автоматически отменяются при очистке ViewModel (onCleared()), предотвращая утечки памяти и лишнюю работу.

<br>

## 3 Какие преимущества даёт использование sealed class для представления состояний UI?
Ограниченный набор состояний (Loading, Success, Error) гарантирует полную обработку всех вариантов в when (компилятор проверит). Каждое состояние может содержать только ему нужные данные, что делает управление UI строгим и безопасным.

<br>

## 4 Как имитировать задержку в корутине?
Вызовом функции delay(миллисекунды). Она приостанавливает корутину, не блокируя поток, и возобновляет её по истечении указанного времени.

<br>

## 5 Как обрабатывать ошибки при выполнении корутин?
Оборачивать код внутри launch (или другого билдера корутин) в блок try-catch. В catch можно перехватить исключение и изменить состояние UI на Error, показав сообщение пользователю.

<br>

## Выводы

В ходе выполнения лабораторной работы я научился использовать viewModelScope для запуска длительных операций в фоновом потоке, управлять состояниями пользовательского интерфейса с помощью sealed class и StateFlow, имитировать задержки с помощью delay(), обрабатывать ошибки в корутинах. 
