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
Научный руководитель<br>
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
листниг ViewModel
(в 7 был реализован parceable dataclass)
<br>


```kotlin
package com.example.myapplication

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow

class MainViewModel : ViewModel() {

    // Приватный изменяемый StateFlow с начальным значением (пустой список)
    private val _tasks = MutableStateFlow<List<Task>>(emptyList())

    // Публичный неизменяемый StateFlow для подписки из UI
    val tasks: StateFlow<List<Task>> = _tasks.asStateFlow()

    // Добавление новой задачи
    fun addTask(task: Task) {
        val currentList = _tasks.value.toMutableList()
        currentList.add(task)
        _tasks.value = currentList
    }

    // Удаление задачи по индексу
    fun deleteTask(index: Int) {
        val currentList = _tasks.value.toMutableList()
        if (index in currentList.indices) {
            currentList.removeAt(index)
            _tasks.value = currentList
        }
    }

    // Обновление текста задачи
    fun updateTask(index: Int, newText: Task) {
        val currentList = _tasks.value.toMutableList()
        if (index in currentList.indices) {
            currentList[index] = newText
            _tasks.value = currentList
        }
    }


}
```


<br>
Листинг main
<br>


```kotlin
package com.example.myapplication
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
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView


import kotlinx.coroutines.launch
class MainActivity : AppCompatActivity() {
    private var doned = 0
    //private val tasks = mutableListOf<Task>()
    private var tasks = emptyList<Task>()
    private lateinit var adapter: TaskAdapter
    private lateinit var  Detailslauncher: ActivityResultLauncher<Intent>

    private val viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val readycount = findViewById<TextView>(R.id.readycount)
        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val editTextheader = findViewById<EditText>(R.id.editTextheader)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)
        val recyclerView = findViewById<RecyclerView>(R.id.recyclerViewTasks)
        doned=viewModel.tasks.value.count{ item -> item.done }
        readycount.text = $"сделано задач ${doned}"

        Detailslauncher  = registerForActivityResult(ActivityResultContracts.StartActivityForResult()){
                result ->
            if (result.resultCode== Activity.RESULT_CANCELED){
                viewModel.deleteTask(result.data?.getIntExtra("pos",0) ?:0)
                //tasks.removeAt(result.data?.getIntExtra("pos", 0) ?: 0)
                adapter.updateData(viewModel.tasks.value)
                doned=viewModel.tasks.value.count{ item -> item.done }
                readycount.text = $"сделано задач ${doned}"
                adapter.notifyItemRemoved(result.data?.getIntExtra("pos", 0) ?: 0)
                Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
            }
            if (result.resultCode == 3){
               // viewModel.deleteTask(result.data?.getIntExtra("pos",0) ?:0)
                //tasks.removeAt(result.data?.getIntExtra("pos", 0) ?: 0)
               val newTask = result.data?.getParcelableExtra<Task>("nw") ?:viewModel.tasks.value[result.data?.getIntExtra("pos", 0) ?: 0]
                viewModel.updateTask(result.data?.getIntExtra("pos", 0) ?: 0,newTask)
                adapter.updateData(viewModel.tasks.value)
               // doned=viewModel.tasks.value.count{ item -> item.done }
               // readycount.text = $"сделано задач ${doned}"
               // adapter.notifyItemRemoved(result.data?.getIntExtra("pos", 0) ?: 0)
               Toast.makeText(this, "Задача изменена", Toast.LENGTH_SHORT).show()
            }
        }


        // Настройка RecyclerView
        recyclerView.layoutManager = LinearLayoutManager(this)
        adapter = TaskAdapter(emptyList(),

            { position ->
                viewModel.deleteTask(position)
              //  tasks.removeAt(position)
                adapter.updateData(viewModel.tasks.value)
                doned=viewModel.tasks.value.count { item -> item.done }
                readycount.text = $"сделано задач ${doned}"
                adapter.notifyItemRemoved(position)
                Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
            },
            ///
            { chk,position->


               viewModel.taskState(position,chk)
                doned=viewModel.tasks.value.count { item -> item.done }
                readycount.text = $"сделано задач ${doned}"},



            {position ->
               // val taskText = tasks[position]
                val intent = Intent(this, DetailActivity::class.java)
                intent.putExtra("task", viewModel.tasks.value[position])
                intent.putExtra("pos",position)

                Detailslauncher.launch(intent)

            })


        recyclerView.adapter = adapter


        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.tasks.collect { tasks ->
                    adapter.updateData(tasks) // предполагаем, что у адаптера есть такой метод
                }
            }
        }
        // Добавление задачи
        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            val header = editTextheader.text.toString()
            if (task.isNotBlank()) {
               // tasks.add(Task(header,task,false))
                viewModel.addTask(Task(header,task,false))
                adapter.notifyItemInserted(tasks.size - 1) // более эффективно, чем notifyDataSetChanged
                editTextTask.text.clear()
                editTextheader.text.clear()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }

        // Восстановление данных при повороте (опционально, см. Лаб.5)
        /*if (savedInstanceState != null) {
            val savedTasks = savedInstanceState.getParcelableArrayList<Task>("tasks"
            )
            if (savedTasks != null) {
               // tasks.clear()
                //tasks.addAll(savedTasks)
                adapter.notifyDataSetChanged()
            }
        }*/
    }

   /* override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)

        outState.putParcelableArrayList("tasks", ArrayList<Task>(tasks))
    }*/
}
```

<br>
Листинг Адаптера
<br>

```kotlin

import android.graphics.Color
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

class TaskAdapter(private var tasks: List<Task>,
                  private val onItemLongClick: (Int) -> Unit,
                  private val onCheck: (Boolean, Int) -> Unit,
                  private val onItemClick: (Int) -> Unit
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
        val taskname = TASK.task
        val header = TASK.header
        holder.textHeader.text = header
        holder.textTask.text = taskname
        holder.checkTask.isChecked=TASK.done
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
            onItemClick(position)
        }

        // Обработка чекбокса (опционально)
        holder.checkTask.setOnCheckedChangeListener { _, isChecked ->
            // Можно добавить логику отметки выполнения, например, перечеркивание текста
            if (isChecked) {

                holder.textTask.paintFlags =
                    holder.textTask.paintFlags or android.graphics.Paint.STRIKE_THRU_TEXT_FLAG
                onCheck(true, position)

            } else {

                holder.textTask.paintFlags = holder.textTask.paintFlags and android.graphics.Paint.STRIKE_THRU_TEXT_FLAG.inv()

                onCheck(false, position)
            }
        }
        holder.itemView.setOnLongClickListener {
            onItemLongClick(position)
            holder.checkTask.isChecked=false
            true
        }

    }

    override fun getItemCount(): Int = tasks.size


    // Метод для обновления списка
    fun updateData(newTasks: List<Task>) {
        //     tasks.clear()
        //   tasks.addAll(newTasks)

        tasks = newTasks
        notifyDataSetChanged()
    }
}
```

<br>
<img width="435" height="874" alt="image" src="https://github.com/user-attachments/assets/172e06d2-f5b6-4bc0-bedb-2a696dc338a0" />
<img width="549" height="275" alt="image" src="https://github.com/user-attachments/assets/f845faac-0839-4e29-9fb9-1138ca2a32d2" />
<br>

## 1 Для чего нужен ViewModel? Как он помогает при повороте экрана?
ViewModel хранит UI-данные и живёт дольше Activity/Fragment.

При повороте Activity уничтожается и создаётся заново, но ViewModel сохраняется, данные не теряются и не загружаются повторно.

Гарантирует автоматическую очистку (onCleared) при финише владельца, предотвращая утечки.
<br>

## 2 Чем StateFlow отличается от LiveData? В каких случаях предпочтительнее использовать StateFlow?
StateFlow и LiveData делают одно и то-же, но StateFlow работает асинхронно, а так - же имеет больше встроенного функционала. StateFlow кроссплатформенный, тогда как LiveData только под андроид
Когда StateFlow предпочтительнее:

Активное использование корутин и Flow.

Сложные реактивные цепочки, комбинирование потоков.

Кроссплатформенная разработка (Kotlin Multiplatform).
<br>

## 3 Что такое lifecycleScope и repeatOnLifecycle? Зачем они нужны при подписке на StateFlow?
lifecycleScope — это область видимости (scope) корутины, привязанная к жизненному циклу компонента (Activity или Fragment). Корутины, запущенные в этом скоупе, автоматически отменяются (cancels), когда компонент уничтожается (onDestroy).
repeatOnLifecycle(state) — это функция-расширение, которая запускает переданный блок кода в новой корутине, когда жизненный цикл достигает нужного состояния (например, Lifecycle.State.STARTED). Как только состояние падает ниже заданного (например, при переходе приложения в фон, когда вызывается onStop), эта внутренняя корутина автоматически отменяется. Когда пользователь возвращается, корутина перезапускается.
<br>

## 4 Как обновить данные в StateFlow?
StateFlow представлен двумя интерфейсами: StateFlow<T> (только для чтения) и MutableStateFlow<T> (для изменения). Для обновления данных используется MutableStateFlow
<br>

```kotlin
private val _state = MutableStateFlow(initialValue)
val state: StateFlow<UiState> = _state

fun updateData(newValue: UiState) {
    _state.value = newValue
}
```

<br>

## 5 Какие преимущества даёт вынос логики в ViewModel с точки зрения тестирования?
ViewModel спроектирована так, что её можно легко тестировать изолированно, без реального Android-окружения. Вся логика (загрузка данных, обработка ошибок, преобразования, управление состоянием UI) сосредоточена в ViewModel. Тесты проверяют именно бизнес-правила, а не взаимодействие с Android-фреймворком.
<br>
Вывод:  изучены архитектурный компонент ViewModel и реактивный поток StateFlow, а также способы их интеграции с жизненным циклом Activity. Практически реализован перенос логики управления списком задач из MainActivity в выделенный класс MainViewModel, что позволило отделить бизнес-логику и состояние UI от слоя представления.
