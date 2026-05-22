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

Лабораторная работа №9
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
Листинг TaskEntity
<br>

```kotlin
package com.example.todoapp.database

import android.os.Parcelable
import androidx.room.Entity
import androidx.room.PrimaryKey
import kotlinx.parcelize.Parcelize

@Entity(tableName = "tasks") // Имя таблицы в БД
@Parcelize
data class TaskEntity(
    @PrimaryKey(autoGenerate = true) // Автоматическая генерация ID
    val id: Long = 0,
    val header: String,
    var title: String,          // Текст задачи
    val isCompleted: Boolean = false, // Статус выполнения
    val createdTime: Long = System.currentTimeMillis() // Время создания для сортировки
) : Parcelable

```

<br>
Листинг TaskDao.kt
<br>

```kotlin

@Dao
interface TaskDao {

    @Query("SELECT * FROM tasks")
    fun getAllTasks(): Flow<List<TaskEntity>> // Возвращаем Flow для реактивного обновления
    @Query("SELECT * FROM tasks WHERE isCompleted = True")
    fun getReady(): Flow<List<TaskEntity>>
    @Query("SELECT * FROM tasks WHERE header=:header")
    suspend fun byheader(header: String): List<TaskEntity>
    @Insert(onConflict = OnConflictStrategy.REPLACE) // При конфликте заменять
    suspend fun insertTask(task: TaskEntity)
    @Query("SELECT * FROM tasks ORDER BY IsCompleted")
    fun byReady(): Flow<List<TaskEntity>>
    @Update
    suspend fun updateTask(task: TaskEntity)

    @Delete
    suspend fun deleteTask(task: TaskEntity)

    @Query("DELETE FROM tasks")
    suspend fun deleteAll()

    @Query("SELECT * FROM tasks WHERE id = :id")
    suspend fun getTaskById(id: Long): TaskEntity?


}
```

<br>
AppDatabase.kt
<br>

```kotlin
package com.example.myapplication.database

import com.example.todoapp.database.TaskDao
import com.example.todoapp.database.TaskEntity
import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(
    entities = [TaskEntity::class],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun taskDao(): TaskDao

    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getInstance(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "todo_database"
                )
                    //.fallbackToDestructiveMigration() // Для разработки: при изменении версии БД пересоздавать таблицы
                    .build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

<br>
MainViewModel.kt
<br>

```kotlin
class MainViewModel (
    private val database: AppDatabase
): ViewModel() {



    private val taskDao = database.taskDao()
    var needed = MutableStateFlow<List<TaskEntity>>(emptyList())

    val tasks: StateFlow<List<TaskEntity>> = taskDao.getAllTasks().stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000),
        initialValue = emptyList()
    )


    fun loadTasksForCategory(category: String) {
        viewModelScope.launch {
            needed.value = taskDao.byheader(category)
        }
    }


    val readyTasks: StateFlow<List<TaskEntity>> = taskDao.getReady()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(6000),
            initialValue = emptyList()
        )

    fun addTask(task: TaskEntity) {
        viewModelScope.launch {
            val task = TaskEntity(title = task.title, header = task.header)
            taskDao.insertTask(task)
        }

    }

    fun deleteTask(task: TaskEntity) {
        viewModelScope.launch {
            taskDao.deleteTask(task)
        }
    }

    fun updateTask(task: TaskEntity) {
        viewModelScope.launch {
            taskDao.updateTask(task)
        }
    }

    fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        viewModelScope.launch {
            val updatedTask = task.copy(isCompleted = isCompleted)
            taskDao.updateTask(updatedTask)
        }
    }

    fun deleteAllTasks() {
        viewModelScope.launch {
            taskDao.deleteAll()
        }
    }


    }


```

<br>
Листинг MainActivity.kt
<br>

```kotlin
import android.app.Activity
import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.activity.result.ActivityResultLauncher
import androidx.activity.result.contract.ActivityResultContracts
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.isVisible
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.myapplication.database.AppDatabase
import com.example.todoapp.database.TaskEntity
import com.google.android.material.snackbar.Snackbar
import kotlinx.coroutines.flow.flowOf


import kotlinx.coroutines.launch
import kotlin.text.clear
import kotlin.toString

class MainActivity : AppCompatActivity() {

    //private val tasks = mutableListOf<Task>()
    private var tasks = emptyList<TaskEntity>()
    private lateinit var adapter: TaskAdapter
    private lateinit var  Detailslauncher: ActivityResultLauncher<Intent>
    var  ready = 0
    class MainViewModelFactory(

        private val database: AppDatabase
    ) : ViewModelProvider.Factory {
        override fun <T : ViewModel> create(modelClass: Class<T>): T {
            if (modelClass.isAssignableFrom(MainViewModel::class.java)) {
                @Suppress("UNCHECKED_CAST")
                return MainViewModel(database) as T
            }
            throw IllegalArgumentException("Unknown ViewModel class")
        }
    }
    private val database by lazy { AppDatabase.getInstance(this) }
    private val viewModel: MainViewModel by viewModels {
        MainViewModelFactory(database)
    }


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val readycount = findViewById<TextView>(R.id.readycount)
        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val editTextheader = findViewById<EditText>(R.id.editTextheader)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)
        val recyclerView = findViewById<RecyclerView>(R.id.recyclerViewTasks)
        val searchbutton = findViewById<Button>(R.id.buttonsearch)
        val searchedd = findViewById<EditText>(R.id.editTextSearch)
        ready = viewModel.readyTasks.value.count()
        readycount.text = $"сделано задач ${viewModel.readyTasks.value.count()}"
            // Создание лаунчера DetailActictivity, с возвратом разных result code
        Detailslauncher  = registerForActivityResult(ActivityResultContracts.StartActivityForResult()){
                result ->
            if (result.resultCode== Activity.RESULT_CANCELED){
                viewModel.deleteTask(result.data?.getParcelableExtra<TaskEntity>("Task") ?: viewModel.tasks.value[0])
                adapter.updateData(viewModel.tasks.value)
                ready = viewModel.readyTasks.value.count()
                readycount.text = $"сделано задач ${viewModel.readyTasks.value.count()}"
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
              //  tasks.removeAt(position)
                adapter.updateData(viewModel.tasks.value)
                ready  = viewModel.readyTasks.value.count()
                readycount.text = $"сделано задач ${viewModel.readyTasks.value.count()}"
                adapter.notifyItemRemoved(position)
                Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
            },
            ///
            { chk,task, done->
                viewModel.toggleTaskCompletion(task,chk)
               // adapter.updateData(tasks)

                readycount.text = $"сделано задач ${done}"},



            {position, Task ->
               // val taskText = tasks[position]
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
                // tasks.add(Task(header,task,false))
                viewModel.addTask(TaskEntity(header=header, title =task))
                //  adapter.notifyItemInserted(tasks.size -1) // более эффективно, чем notifyDataSetChanged
                //adapter.notifyDataSetChanged()
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
        }
```

<br>
TaskAdapter.kt
<br>

```kotlin
import android.graphics.Color
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.example.todoapp.database.TaskEntity

class TaskAdapter(private var tasks: List<TaskEntity>,
                  private val onItemLongClick: (Int, TaskEntity) -> Unit,
                  private val onCheck: (Boolean,TaskEntity, Int) -> Unit,
                  private val onItemClick: (Int, TaskEntity) -> Unit
) :
    RecyclerView.Adapter<TaskAdapter.TaskViewHolder>() {


    // ViewHolder хранит ссылки на элементы внутри карточки
    class TaskViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textHeader: TextView = itemView.findViewById(R.id.TaskHeader)
        val textTask: TextView = itemView.findViewById(R.id.textTask)
        val checkTask: CheckBox = itemView.findViewById(R.id.checkTask)

    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TaskViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_task, parent, false)
        val hldr = TaskViewHolder(view)

        return hldr
    }

    override fun onBindViewHolder(holder: TaskViewHolder, position: Int) {

        var TASK =  tasks[position]
        val taskname = TASK.title
        val header = TASK.header
        val done  = tasks.count{item -> item.isCompleted}
        holder.textHeader.text = header
        holder.textTask.text = taskname
        holder.checkTask.isChecked=TASK.isCompleted
        if (position % 2 == 0) {
            // Четная позиция
           holder.textTask.setBackgroundResource(R.color.EvenlyRed)
           holder.itemView.setBackgroundResource(R.color.EvenlyBlue)
            holder.textTask.setTextColor(Color.WHITE)

        } else {
            // Нечетная позиция
            holder.textTask.setBackgroundResource(R.color.UnevenlyGreen)
            holder.textTask.setTextColor(Color.BLACK)
           holder.itemView.setBackgroundResource(R.color.UnvenlyPurple)
        }


        holder.itemView.setOnClickListener {
            onItemClick(position,TASK)
        }

        // Обработка чекбокса (опционально)
        holder.checkTask.setOnCheckedChangeListener { _, isChecked ->
            // Можно добавить логику отметки выполнения, например, перечеркивание текста
            if (isChecked) {

                holder.textTask.paintFlags =
                    holder.textTask.paintFlags or android.graphics.Paint.STRIKE_THRU_TEXT_FLAG
                onCheck(true, TASK, done+1)

            } else {

                holder.textTask.paintFlags =
                    holder.textTask.paintFlags and android.graphics.Paint.STRIKE_THRU_TEXT_FLAG.inv()
                onCheck(false, TASK, done-1)
            }
        }

        holder.itemView.setOnLongClickListener {
            onItemLongClick(position, TASK)
            holder.checkTask.isChecked=false
            true
        }

    }

    override fun getItemCount(): Int = tasks.size


    // Метод для обновления списка
    fun updateData(newTasks: List<TaskEntity>) {
        //     tasks.clear()
        //   tasks.addAll(newTasks)

        tasks = newTasks
       // tasks = tasks.reversed()
        notifyDataSetChanged()
    }
}
```

<br>
<img width="302" height="599" alt="image" src="https://github.com/user-attachments/assets/b7e07c69-e4c3-4af9-9b5f-465036e78500" />
<img width="295" height="601" alt="image" src="https://github.com/user-attachments/assets/892d4c25-f5cd-4ff1-91e4-1662878bd13d" />

<br>
<br>

## 1 Для чего нужна библиотека Room? Какие проблемы она решает по сравнению с прямым использованием SQLite?
Room — это уровень абстракции над SQLite, входящий в состав Android Jetpack. Она нужна, чтобы упростить работу с локальной базой данных и избавить разработчика от типичных проблем прямого использования SQLite.
Проблемы, решаемые Room:
Отсутствие проверки SQL на этапе компиляции – в сыром SQLite ошибки в запросах обнаруживаются только во время выполнения, Room проверяет корректность SQL-запросов при сборке.
Ручное преобразование данных – Room автоматически сопоставляет строки таблиц с объектами Kotlin/Java (Entity), избавляя от написания шаблонного кода.

## 2. Назовите три основных компонента Room и объясните их назначение
Entity
Класс, аннотированный @Entity, представляет таблицу в базе данных. Каждое поле класса становится столбцом, а сам объект — строкой таблицы. Entity определяет схему данных.
DAO (Data Access Object)
Интерфейс или абстрактный класс, помеченный @Dao, содержит методы для доступа к данным. Здесь объявляются SQL-запросы (через @Query, @Insert, @Update, @Delete). Room генерирует реализацию DAO во время компиляции.
Database
Абстрактный класс, наследующий RoomDatabase, аннотированный @Database. Связывает все Entity и DAO, служит точкой входа для создания экземпляра базы данных. Содержит список сущностей и номер версии.

## 3. Почему методы DAO, изменяющие данные, объявляются как suspend?
Room требует, чтобы операции, изменяющие данные (INSERT, UPDATE, DELETE), выполнялись вне главного потока, чтобы не блокировать UI. Объявление метода как suspend заставляет вызывающий код использовать корутину (или другой suspend-контекст), и Room автоматически запускает эту операцию на фоновом потоке, указанном в RoomDatabase.Builder (по умолчанию — Architecture Components I/O Executor). Это гарантирует безопасное выполнение без явного переключения потоков вручную.
Для методов, возвращающих Flow или LiveData, Room сам управляет потоками, поэтому их не нужно помечать suspend.

## 4. Что такое Flow и почему его удобно использовать с Room?
Flow — это асинхронный поток данных из библиотеки Kotlin Coroutines, который испускает значения последовательно. Он поддерживает реактивное программирование: подписчики автоматически получают новые данные при их появлении.
Удобство с Room:
Room возвращает Flow<List<Entity>> для запросов @Query. При любом изменении данных в отслеживаемых таблицах Room автоматически выполняет запрос заново и обновляет Flow новым результатом.
Это позволяет строить реактивные UI: в Compose достаточно подписаться через collectAsState(), и интерфейс будет перерисовываться при изменении базы данных без явных повторных запросов.
Flow работает с корутинами, легко комбинируется с другими асинхронными операциями и поддерживает операторы преобразования (map, filter, flatMapLatest и др.).
    
## 5. Как Room обеспечивает проверку SQL-запросов на этапе компиляции?
Room использует аннотационную обработку (KAPT/KSP) во время компиляции. Для каждого метода DAO, содержащего @Query, Room:
Анализирует строку SQL-запроса и сверяет имена таблиц и столбцов с существующими Entity.
Проверяет типы параметров и возвращаемого значения на соответствие столбцам.
Генерирует код, который будет выполнять этот запрос.
Если в SQL-запросе есть синтаксическая ошибка, несуществующая таблица или столбец, сборка завершится с ошибкой, а разработчик увидит конкретное сообщение. Таким образом, многие ошибки отлавливаются на этапе написания кода, а не во время выполнения.
    
## 6. Зачем нужен паттерн Singleton для экземпляра базы данных?
Создание нескольких экземпляров RoomDatabase может привести к утечкам памяти и несогласованному состоянию, поскольку каждый экземпляр открывает собственное подключение к базе. Обычно применяют паттерн Singleton, чтобы:
Во всём приложении существовал только один экземпляр базы данных.
Гарантировать потокобезопасный доступ: RoomDatabase уже потокобезопасен, но единственный экземпляр упрощает управление.
Экономить ресурсы, переиспользуя одно соединение с БД вместо многократного открытия/закрытия.
Избежать конфликтов параллельных миграций и блокировок.
Типичная реализация в Android использует companion object с @Volatile переменной и синхронизацией для ленивой инициализации.

## Выводы
Были освоены методы доступа к базе данных через Room
