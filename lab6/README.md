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


Листинг файла item_task.xml.


<br>

```xml

<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"

    app:cardCornerRadius="8dp"
    app:cardElevation="4dp"
    app:cardBackgroundColor="#FFFFFF">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="16dp">

        <TextView
            android:id="@+id/TaskHeader"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:textSize="18sp"
            android:textColor="#333333"/>
        <TextView
            android:id="@+id/textTask"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:textSize="18sp"
            android:textColor="#333333"/>

        <CheckBox
            android:id="@+id/checkTask"
            android:checked="false"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>


    </LinearLayout>

</androidx.cardview.widget.CardView>

```


<br>

Листинг класса TaskAdapter.


```kotlin

package com.example.myapplication123123123123

import android.graphics.Color
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

class TaskAdapter(private  var taskheaders: MutableList<String>, private var tasks: MutableList<String>,
                  private val onItemLongClick: (Int) -> Unit,
                  private val onCheck: (Boolean) -> Unit) :
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
        val task = tasks[position]
        val header = taskheaders[position]

        holder.textHeader.text = header
        holder.textTask.text = task
        if (holder.checkTask.isChecked){
        holder.checkTask.isChecked = false
        }
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
        // Обработка чекбокса (опционально)
        holder.checkTask.setOnCheckedChangeListener { _, isChecked ->
            // Можно добавить логику отметки выполнения, например, перечеркивание текста
            if (isChecked) {
                holder.textTask.paintFlags = holder.textTask.paintFlags or android.graphics.Paint.STRIKE_THRU_TEXT_FLAG
                onCheck(isChecked)

            } else {
                holder.textTask.paintFlags = holder.textTask.paintFlags and android.graphics.Paint.STRIKE_THRU_TEXT_FLAG.inv()

                onCheck(isChecked)
            }
        }
        holder.itemView.setOnLongClickListener {
            onItemLongClick(position)
            true
        }

    }

    override fun getItemCount(): Int = tasks.size


    // Метод для обновления списка
    fun updateData(newHeades: List<String>, newTasks: List<String>) {
    //     tasks.clear()
     //   tasks.addAll(newTasks)
        taskheaders = newHeades as MutableList<String>
        tasks = newTasks as MutableList<String>
        notifyDataSetChanged()
    }
}

```



<br>
Листинг MainActivity.kt с изменениями.
<br>


```kotlin

package com.example.myapplication123123123123
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {
    private var done = 0
    private val tasks = mutableListOf<String>()
    private val tasksHeaders = mutableListOf<String>()
    private lateinit var adapter: TaskAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val readycount = findViewById<TextView>(R.id.readycount)
        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val editTextheader = findViewById<EditText>(R.id.editTextheader)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)
        val recyclerView = findViewById<RecyclerView>(R.id.recyclerViewTasks)

        // Настройка RecyclerView
        recyclerView.layoutManager = LinearLayoutManager(this)
        adapter = TaskAdapter(tasksHeaders,tasks,
            { position ->
                tasksHeaders.removeAt(position)
                tasks.removeAt(position)
                adapter.updateData(tasksHeaders,tasks)
                if(done>=1){
                done -=1}
                readycount.text = $"сделано задач ${done}"
            adapter.notifyItemRemoved(position)
        },{chkd -> if(chkd){

            done+=1

        }else{
            if (done>=1)
                done -=1
        }
                readycount.text = $"сделано задач ${done}"})
        recyclerView.adapter = adapter

        // Добавление задачи
        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            val header = editTextheader.text.toString()
            if (task.isNotBlank()) {
                tasks.add(task)
                tasksHeaders.add(header)
                adapter.notifyItemInserted(tasks.size - 1) // более эффективно, чем notifyDataSetChanged
                editTextTask.text.clear()
                editTextheader.text.clear()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }

        // Восстановление данных при повороте (опционально, см. Лаб.5)
        if (savedInstanceState != null) {
            val savedTasks = savedInstanceState.getStringArrayList("tasks")
            val savedheaders = savedInstanceState.getStringArrayList("headers")
            if (savedTasks != null) {
                tasks.clear()
                tasks.addAll(savedTasks)
                tasksHeaders.clear()
                if (savedheaders != null) {
                    tasksHeaders.addAll(savedheaders)
                }
                adapter.notifyDataSetChanged()
            }
        }
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putStringArrayList("headers", ArrayList(tasksHeaders))
        outState.putStringArrayList("tasks", ArrayList(tasks))
    }
}

```
<br>


Скриншот работающего приложения с несколькими карточками.
<img width="720" height="1440" alt="image" src="https://github.com/user-attachments/assets/31e18635-bc91-464f-81ae-42e45610a411" />


<br>

Ответы на контрольные вопросы
1 Для чего нужен RecyclerView? Чем он лучше ListView?

RecyclerView — это гибкий и производительный контейнер для отображения больших наборов данных в виде списка, сетки или кастомной компоновки. Он был создан как современная замена ListView.
Лучше за счёт оптимизации - ячейки переиспользуются,
можно обновлять данные только частично за счёт частичных уведомлений  (notifyItemInserted, notifyItemChanged с полезной нагрузкой и т.д.)
Анимации из коробки (item animator)
<br>
2 Какие компоненты необходимы для работы RecyclerView?
Обязательные компоненты:

Сам RecyclerView в разметке.

Адаптер (наследник RecyclerView.Adapter) — связывает данные с визуальными элементами, создаёт и заполняет ViewHolder.

ViewHolder — хранит ссылки на элементы интерфейса для каждой ячейки, позволяет избежать многократного поиска по id.

LayoutManager — определяет, как элементы располагаются на экране (список, сетка и т.д.). Без него RecyclerView не отобразит данные.

Дополнительные (опционально):

ItemDecoration — отступы, разделители, декоративные элементы.

ItemAnimator — анимации добавления, удаления, перемещения.

ItemTouchHelper — обработка свайпов и перетаскивания.
<br>
3. Что такое ViewHolder и для чего он используется?
ViewHolder — объект, кеширующий ссылки на View-элементы одной ячейки.
Назначение: избежать повторных вызовов findViewById() при переиспользовании макета, что даёт прирост производительности и экономию памяти.
<br>
4. Чем отличается notifyDataSetChanged() от notifyItemInserted()?
notifyDataSetChanged() — полное обновление всех видимых элементов, без анимаций. Тяжёлая операция.

notifyItemInserted(pos) — вставка одного элемента с анимацией, перерисовываются только затронутые элементы. Лучше для точечных изменений.

<br>

5. Как добавить обработку кликов на элементы RecyclerView?
Передайте лямбду или интерфейс в адаптер. В onBindViewHolder() установите setOnClickListener на корневое View и вызывайте переданный колбэк, пробрасывая данные или позицию.
Пример: holder.itemView.setOnClickListener { onItemClick(item) }.
Для кликов по отдельным элементам ячейки — аналогично, но слушатель вешается на конкретную View внутри ViewHolder.

<br>

Выводы: был освоен современный способ представления массивов данных
