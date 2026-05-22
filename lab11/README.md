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

Лабораторная работа №11
Интеграция Room в проект. Сохранение списка задач в БД
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
листинг TaskRepository

<br>

```kotlin

import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.flow.Flow

interface TaskRepository {
    fun getAllTasks(): Flow<List<TaskEntity>>
    suspend fun addTask(task: TaskEntity): Boolean
    suspend fun addTasktst(task: TaskEntity): Boolean
    suspend fun deleteTask(task: TaskEntity): Boolean
    suspend fun updateTask(task: TaskEntity) : Boolean
    suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) : Boolean
    suspend fun deleteAllTasks() : Boolean
    suspend fun loadTaskByHeader(header: String): Pair<List<TaskEntity>, Boolean>
}
```

<br>
Листинг TaskRepositoryImpl
<br>

```kotlin
class TaskRepositoryImpl(
    private val taskDao: TaskDao
) : TaskRepository {

    override fun getAllTasks(): Flow<List<TaskEntity>> = taskDao.getAllTasks()

    override suspend fun addTask(task: TaskEntity): Boolean {
        runCatching{ val task = TaskEntity(title = task.title, header =task.header)
            taskDao.insertTask(task)
            return true
        }.onFailure { return false }
        return false
    }
   override  suspend fun addTasktst(task: TaskEntity): Boolean {
        val task = TaskEntity(id = task.id,title = task.title, header =task.header)
       runCatching{taskDao.insertTask(task)
           return true}.onFailure{return false}
       return false
    }

    override suspend fun deleteTask(task: TaskEntity): Boolean {

            runCatching{taskDao.deleteTask(task)
                return true}.onFailure { return false }
        return false
        }



    override suspend fun updateTask(task: TaskEntity): Boolean {

            runCatching{taskDao.updateTask(task)
                return true}.onFailure { return false }
        return false
    }

    override suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean): Boolean {
       runCatching {  val updatedTask = task.copy(isCompleted = isCompleted)
        taskDao.updateTask(updatedTask)
           return true  }.onFailure { return false }
        return false
        }


    override suspend fun deleteAllTasks(): Boolean {

            runCatching {taskDao.deleteAll()
                return true
        }.onFailure { return false }
        return false
    }

    override suspend fun loadTaskByHeader(header: String): Pair<List<TaskEntity>,Boolean> {
       runCatching {
           val lst = taskDao.byheader(header)
       return Pair(lst,true)
       }.onFailure {return Pair(emptyList(),false)  }
        return Pair(emptyList(),true)
    }
}
```
<br>
Листинг MainViewModel
<br>

```kotlin

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.myapplication.data.TaskRepository
import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class MainViewModel (
    private val repository: TaskRepository
): ViewModel() {



    private  var  _mistake = MutableStateFlow<Boolean>(true)
    val mistake: StateFlow<Boolean> = _mistake.asStateFlow()
    var needed = MutableStateFlow<List<TaskEntity>>(emptyList())

    val tasks: StateFlow<List<TaskEntity>> = repository.getAllTasks().stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(3000),

        initialValue = emptyList()
    )
    fun loadTasksForCategory(category: String) {
        viewModelScope.launch{
            val par = repository.loadTaskByHeader(category)
            needed.value = par.first
            _mistake.value = par.second
        }
    }
    fun reloadMistakes(){
        _mistake.value = true
    }

    fun addTask(task: TaskEntity) {
        viewModelScope.launch(Dispatchers.Main.immediate) {
            val task = TaskEntity(title = task.title, header = task.header)
           _mistake.value = repository.addTask(task)
        }

    }

    fun addTasktst(task: TaskEntity) {
        viewModelScope.launch {
            val data =  repository.addTasktst(task)
            _mistake.value = data
        }

    }


    fun deleteTask(task: TaskEntity){
       viewModelScope.launch{
           _mistake.value = repository.deleteTask(task)
        }
    }

    fun updateTask(task: TaskEntity) {
        viewModelScope.launch {
            _mistake.value =  repository.updateTask(task)
        }
    }

    fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        viewModelScope.launch{
            val updatedTask = task.copy(isCompleted = isCompleted)
            _mistake.value =  repository.updateTask(updatedTask)
        }
    }

    fun deleteAllTasks() {
        viewModelScope.launch {
            _mistake.value =  repository.deleteAllTasks()
        }
    }
    }
```

<br>
Листинг MainViewModelFactory
<br>

```kotlin
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
```

<br>
Листинг MainActivity
<br>

```kotlin
package com.example.myapplication
import android.app.Activity
import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
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
import com.example.todoapp.database.TaskEntity
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
        val mbutton = findViewById<Button>(R.id.button)
            // Создание лаунчера DetailActictivity, с возвратом разных result code
        Detailslauncher  = registerForActivityResult(ActivityResultContracts.StartActivityForResult()){
                result ->
            if (result.resultCode== RESULT_CANCELED){
                viewModel.deleteTask(result.data?.getParcelableExtra<TaskEntity>("Task") ?: viewModel.tasks.value[0])
                adapter.updateData(viewModel.tasks.value)
                if (!viewModel.mistake.value){
                    Toast.makeText(this, "Что то пошло не так....", Toast.LENGTH_SHORT).show()
                    viewModel.reloadMistakes()
                }

                adapter.notifyItemRemoved(result.data?.getIntExtra("pos", 0) ?: 0)
                Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
            }
            if (result.resultCode == 3){
               val newTask = result.data?.getParcelableExtra<TaskEntity>("Task") ?:viewModel.tasks.value[result.data?.getIntExtra("pos", 0) ?: 0]
                newTask.title = result.data?.getStringExtra("nw") ?: ""
                viewModel.updateTask(newTask)
                if (!viewModel.mistake.value){
                    Toast.makeText(this, "Что то пошло не так....", Toast.LENGTH_SHORT).show()
                    viewModel.reloadMistakes()
                }
                adapter.updateData(viewModel.tasks.value)
               Toast.makeText(this, "Задача изменена", Toast.LENGTH_SHORT).show()
            }
        }


        // Настройка RecyclerView
        recyclerView.layoutManager = LinearLayoutManager(this)

        adapter = TaskAdapter(tasks,

            { position,task ->
                viewModel.deleteTask(task)
                if (!viewModel.mistake.value){
                    Toast.makeText(this, "Что то пошло не так....", Toast.LENGTH_SHORT).show()
                    viewModel.reloadMistakes()
                }
                adapter.updateData(viewModel.tasks.value)
                adapter.notifyItemRemoved(position)
                Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
            },
            ///
            { chk,task, done->
                viewModel.toggleTaskCompletion(task,chk)
                if (!viewModel.mistake.value){
                    Toast.makeText(this, "Что то пошло не так....", Toast.LENGTH_SHORT).show()
                    viewModel.reloadMistakes()
                }
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

        // Добавление задачи
        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            val header = editTextheader.text.toString()
            if (task.isNotBlank()) {
                viewModel.addTask(TaskEntity(header=header, title =task))
                if (!viewModel.mistake.value){
                    Toast.makeText(this, "Что то пошло не так....", Toast.LENGTH_SHORT).show()
                    viewModel.reloadMistakes()
                }
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
                if (!viewModel.mistake.value){
                    Toast.makeText(this, "Что то пошло не так....", Toast.LENGTH_SHORT).show()
                    viewModel.reloadMistakes()
                }
                tasks = viewModel.needed.value
                for ( t in tasks){
                    Toast.makeText(this, t.title, Toast.LENGTH_SHORT).show()
                }
                searchedd.text.clear()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }
        var mstk = true
        mbutton.setOnClickListener {
            viewModel.addTasktst(TaskEntity(
                id = 1,
                header = "test",
                title = "Tsee",
                isCompleted = true,
                createdTime = System.currentTimeMillis()))

            if (!viewModel.mistake.value){
                Toast.makeText(this,"Что-то Пошло не так", Toast.LENGTH_SHORT).show()
                viewModel.reloadMistakes()
            }

        }

        }
    }
```


<br>
<img width="1889" height="937" alt="image" src="https://github.com/user-attachments/assets/5018e1b4-fdaf-423e-aac1-5e4c8c0f2174" />

<img width="1885" height="878" alt="image" src="https://github.com/user-attachments/assets/dee655d8-50ad-46f1-a7b9-f7c29cf20f94" />

<img width="1878" height="975" alt="image" src="https://github.com/user-attachments/assets/739fe467-9f95-4839-9f5d-3c1335a3974f" />

<bt>
    
Контрольные воросы

<br>

## 1 Какую роль выполняет слой Repository в архитектуре приложения?
Слой Repository (репозиторий) является промежуточным звеном между источниками данных (локальная БД, сеть, кэш) и потребителями данных (ViewModel, другие слои)

## 2 Какие преимущества даёт использование Repository по сравнению с прямым обращением к DAO из ViewModel?

- Слабая связанность (Loose Coupling) – ViewModel не зависит от конкретной реализации базы данных (Room). При замене источника данных (например, переход на другую БД) изменяется только репозиторий, а ViewModel остаётся нетронутой.

- Тестируемость – ViewModel, работающую напрямую с DAO, сложно протестировать без реальной БД. С репозиторием достаточно создать его заглушку и проверить логику ViewModel изолированно.

## 3 Как изменится ViewModel, если мы захотим добавить ещё один источник данных (например, сетевое API)?

ViewModel не изменится, если репозиторий корректно спроектирован. Это одно из главных преимуществ паттерна.

## 4 Почему методы репозитория объявлены как suspend?

- Room требует фонового выполнения – запросы к БД не должны блокировать главный поток (иначе интерфейс зависнет). Room автоматически запускает suspend-функции DAO на фоновом пуле потоков.

- Удобство для вызывающего кода – ViewModel может вызвать repository.insert(task) из корутины (viewModelScope.launch) и не заботиться о переключении потоков вручную.

- Правильная отмена операций – при отмене корутины (например, если пользователь ушёл с экрана) работа suspend-функции прерывается, что предотвращает утечки и лишнюю работу.
## 5 Что такое инверсия зависимостей и как она применяется в данном рефакторинге?
Инверсия зависимостей (Dependency Inversion Principle, DIP) – один из принципов SOLID, гласящий:

1. Модули верхнего уровня не должны зависеть от модулей нижнего уровня. Оба должны зависеть от абстракций.

2. Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.

В контексте нашего рефакторинга:

-До добавления репозитория: ViewModel напрямую зависела от TaskDao – конкретной реализации доступа к БД Room. Это нарушало DIP, так как модуль верхнего уровня (ViewModel) зависел от деталей (конкретного DAO).

-После добавления репозитория: ViewModel зависит от абстракции – интерфейса или класса TaskRepository, который сам по себе является абстракцией над источником данных. TaskRepository может использовать TaskDao, сетевой сервис или что угодно, но ViewModel этого не видит.

<br>

Выводы - освоенны некоторые базовые архитектурные элементы, провели рефактор готового приложения 
