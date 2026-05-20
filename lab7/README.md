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

Лабораторная работа №6
Отображение списка задач из предыдущей лабораторной в красивых карточках
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

Цель работы: Научиться использовать RecyclerView для отображения списка данных, освоить создание адаптера и ViewHolder, применить CardView для оформления элементов списка.
<br>
Листинг acrtivitydetail.xml
<br>
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".DetailActivity">

    <LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Детали задачи"
            android:textSize="24sp"
            android:textStyle="bold"
            android:layout_marginBottom="24sp"/>
        <TextView
            android:id="@+id/textTaskheader"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="18sp"
            android:layout_marginBottom="16sp"/>
        <TextView
            android:id="@+id/textTaskDetail"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="18sp"
            android:layout_marginBottom="16sp"/>
        <TextView
            android:id="@+id/textTaskreadyness"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="18sp"
            android:layout_marginBottom="16sp"/>

        <Button
            android:id="@+id/buttonBack"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Назад"
            android:layout_gravity="center_horizontal"/>

        <Button
            android:id="@+id/buttonDelete"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Удалить"
            android:layout_gravity="center_horizontal"/>
    </LinearLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```
<br>
Листинг activitydetail.kt
<br>

```kotlin
package com.example.myapplication123123123123


import android.app.Activity
import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity

class DetailActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_detail)

        val textTaskDetail = findViewById<TextView>(R.id.textTaskDetail)
        val buttonBack = findViewById<Button>(R.id.buttonBack)
        val buttondel = findViewById<Button>(R.id.buttonDelete)
        val textTaskheader = findViewById<TextView>(R.id.textTaskheader)
        val textTaskreadyness = findViewById<TextView>(R.id.textTaskreadyness)
        // Получаем данные из Intent
        val taskText = intent.getParcelableExtra<Task>("task") ?: "Нет данных"
        val pos =  intent.getIntExtra("pos", 0)
        textTaskDetail.text = (taskText as Task).task
        textTaskheader.text = (taskText as Task).header
        textTaskreadyness.text = if((taskText as Task).done) "Готово" else "Не готово"

        buttonBack.setOnClickListener {
            setResult(Activity.RESULT_OK, intent)
            finish() // закрывает текущую активность и возвращает к предыдущей
        }
        buttondel.setOnClickListener {
            setResult(Activity.RESULT_CANCELED, intent)
            finish()
        }
    }
}
```

<br>
Листинг обновлённого adapter
<br>
```kotlin
package com.example.myapplication123123123123

import android.graphics.Color
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

class TaskAdapter(private var tasks: MutableList<Task>,
                  private val onItemLongClick: (Int) -> Unit,
                  private val onCheck: () -> Unit,
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
                    tasks[position].done = holder.checkTask.isChecked
                   // notifyItemChanged(position)
                    onCheck()

                } else {
                    holder.textTask.paintFlags = holder.textTask.paintFlags and android.graphics.Paint.STRIKE_THRU_TEXT_FLAG.inv()
                    tasks[position].done = holder.checkTask.isChecked

                    //notifyItemChanged(position)
                    onCheck()
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

        tasks = newTasks as MutableList<Task>
        notifyDataSetChanged()
    }
}
```
<br>
activitymain.kt
<br>
```kotlin
package com.example.myapplication123123123123
import android.app.Activity
import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.activity.result.ActivityResultLauncher
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {
    private var doned = 0
    private val tasks = mutableListOf<Task>()
    private lateinit var adapter: TaskAdapter
    private lateinit var  Detailslauncher: ActivityResultLauncher<Intent>



    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val readycount = findViewById<TextView>(R.id.readycount)
        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val editTextheader = findViewById<EditText>(R.id.editTextheader)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)
        val recyclerView = findViewById<RecyclerView>(R.id.recyclerViewTasks)



        Detailslauncher  = registerForActivityResult(ActivityResultContracts.StartActivityForResult()){
                result ->
            if (result.resultCode== Activity.RESULT_CANCELED){
                tasks.removeAt(result.data?.getIntExtra("pos", 0) ?: 0)
               // tasks.removeAt(result.data?.getIntExtra("pos", 0) ?: 0)
                adapter.updateData(tasks)
                doned=tasks.count { item -> item.done }
                readycount.text = $"сделано задач ${doned}"
                adapter.notifyItemRemoved(result.data?.getIntExtra("pos", 0) ?: 0)
            }
        }


        // Настройка RecyclerView
        recyclerView.layoutManager = LinearLayoutManager(this)
        adapter = TaskAdapter(tasks,
            { position ->

                tasks.removeAt(position)
                adapter.updateData(tasks)

                readycount.text = $"сделано задач ${doned}"
            adapter.notifyItemRemoved(position)
        },
            { ->

                doned=tasks.count { item -> item.done }
                readycount.text = $"сделано задач ${doned}"},
            {position ->
               val taskText = tasks[position]
                val intent = Intent(this, DetailActivity::class.java)
                 intent.putExtra("task", tasks[position])
                 intent.putExtra("pos",position)

                    Detailslauncher.launch(intent)

                })
        recyclerView.adapter = adapter

        // Добавление задачи
        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            val header = editTextheader.text.toString()
            if (task.isNotBlank()) {
                tasks.add(Task(header,task,false))
                adapter.notifyItemInserted(tasks.size - 1) // более эффективно, чем notifyDataSetChanged
                editTextTask.text.clear()
                editTextheader.text.clear()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }

        // Восстановление данных при повороте (опционально, см. Лаб.5)
        if (savedInstanceState != null) {
            val savedTasks = savedInstanceState.getParcelableArrayList<Task>("tasks"
            )
            if (savedTasks != null) {
                tasks.clear()
                tasks.addAll(savedTasks)
                adapter.notifyDataSetChanged()
            }
        }
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)

        outState.putParcelableArrayList("tasks", ArrayList<Task>(tasks))
    }
}
```
<br>
<br>
<img width="451" height="880" alt="image" src="https://github.com/user-attachments/assets/6d44de3a-8abd-45ba-a878-170af1abd486" />
<img width="438" height="820" alt="image" src="https://github.com/user-attachments/assets/1ae0550c-f4ec-4fca-861d-2868d53bd5dd" />
<br>
Ответы на вопросы
## 1. Что такое Intent? Какие виды Intent существуют?

- **Intent ** —  – это объект для обмена сообщениями между компонентами Android (Activity, Service, BroadcastReceiver). С его помощью запускают экраны, службы, отправляют широковещательные сообщения, открывают веб-страницы и т.д. Intent содержит описание требуемого действия (action), данные (data), категорию, флаги и дополнительные параметры (extras).
- Основные виды:

Явный (Explicit) Intent – точно указывает, какой компонент нужно запустить, по имени класса.
val intent = Intent(this, SecondActivity::class.java)

Неявный (Implicit) Intent – описывает, что нужно сделать (например, «посмотреть картинку», «отправить текст»), а система сама находит подходящее приложение.
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://example.com"))
<br>

## 2. Как передать данные из одной Activity в другую?
Через Intent extras – пары «ключ-значение», добавляемые методом putExtra()
Для сложных объектов используют Parcelable (рекомендуется) или Serializable. Можно обернуть данные в Bundle и положить его в Intent. Альтернативные способы: SharedViewModel (при навигации внутри одного приложения) или Safe Args в Navigation Component.
  
<br>
## 3.Какие способы обработки кликов на элементах RecyclerView вы знаете?
Интерфейс обратного вызова (listener): Создаётся интерфейс OnItemClickListener, передаётся в адаптер. Во ViewHolder устанавливается setOnClickListener, который вызывает метод интерфейса. Activity/Fragment реализует этот интерфейс.

Лямбда-функция: Передать лямбду (position: Int) -> Unit в адаптер, вызывать её при клике.

## 4.Как создать новую Activity в Android Studio?

Автоматически (рекомендуется):
File → New → Activity → выбрать шаблон (Empty Activity и др.). Указать имя Activity, имя layout-файла и т.д. Android Studio сама создаст класс, XML-разметку и добавит запись в AndroidManifest.xml.

Вручную:

Создать Kotlin/Java-класс, унаследованный от AppCompatActivity (или Activity).
Переопределить onCreate(savedInstanceState: Bundle?), установить разметку через setContentView(R.layout.activity_имя).
Создать layout-файл в res/layout.
Зарегистрировать Activity в манифесте внутри тега <application>:

## 5. Для чего используется метод finish()?

finish() завершает текущую Activity – удаляет её из стека задач, и управление возвращается к предыдущей Activity (или на рабочий стол, если это была единственная Activity). Вызывается системой метод onDestroy(), и Activity уничтожается. 

<br>
Вывод:
Были получены навыки работы с Activities в андроид. Разобраны способы передачи данных между компонентами программы
