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

Лабораторная работа №5
Счетчик нажатий, поле ввода и отображение текста. Реализация ToDo-списка
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
Цель работы: Научиться обрабатывать пользовательский ввод, работать с состоянием (счетчик, список задач), динамически обновлять интерфейс приложения на Kotlin.
Листинг MainActivity.xml
<br>

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <!-- Блок 1: Счётчик -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <TextView
            android:id="@+id/textCounter"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            android:text="@string/counter_text"
            android:textSize="24sp" />

        <TextView
            android:id="@+id/textTotalCounter"
            android:layout_width="245dp"
            android:layout_height="match_parent"
            android:layout_marginBottom="16dp"
            android:text="@string/counter_total_text"
            android:textSize="24sp" />

    </LinearLayout>

    <!-- Блок 2: Поле ввода и отображение текста -->

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="49dp"
        android:orientation="horizontal">

        <Button
            android:id="@+id/buttonIncrement"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="2dp"
            android:text="@string/button_increment" />

        <Button
            android:id="@+id/buttonZero"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/Zero_counter" />
    </LinearLayout>

    <EditText
        android:id="@+id/editTextInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/hint_input"
        android:inputType="text"
        android:layout_marginBottom="8dp"/>

    <Button
        android:id="@+id/buttonShow"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_show"
        android:layout_marginBottom="8dp"/>

    <TextView
        android:id="@+id/textEntered"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/label_entered"
        android:textSize="18sp"
        android:layout_marginBottom="24dp"/>

    <!-- Блок 3: ToDo список -->
    <EditText
        android:id="@+id/editTextTask"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/hint_input"
        android:inputType="text"
        android:layout_marginBottom="8dp"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <Button
            android:id="@+id/buttonAddTask"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="8dp"
            android:text="@string/button_add_task" />

        <Button
            android:id="@+id/ButtonRemLast"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_weight="0"
            android:text="@string/remove_last" />

    </LinearLayout>

    <Button
        android:id="@+id/ZeroTasks"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/Zero_tasks" />

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

            <TextView
                android:id="@+id/textTasks"
                android:layout_width="match_parent"
                android:layout_height="277dp"
                android:layout_weight="1"
                android:background="#F0F0F0"
                android:maxLines="10"
                android:padding="8dp"
                android:scrollbarAlwaysDrawHorizontalTrack="false"
                android:scrollbarAlwaysDrawVerticalTrack="true"

                android:scrollbars="horizontal|vertical"
                android:text="@string/label_tasks"
                android:textSize="18sp" />
        </LinearLayout>
    </ScrollView>

</LinearLayout>
```
<br>
Листинг MainActivity.kt
<br>


```kotlin

package com.example.myapplication

import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat

class MainActivity : AppCompatActivity() {
    private var counter = 0
    private var tcounter=0
    private var tasks = mutableListOf<String>()

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putInt("counter", counter)
        outState.putInt("tcounter", tcounter)
        outState.putStringArrayList("tasks", ArrayList(tasks))
    }

    override fun onRestoreInstanceState(savedInstanceState: Bundle) {
        super.onRestoreInstanceState(savedInstanceState)
        counter = savedInstanceState.getInt("counter")
        tcounter = savedInstanceState.getInt("tcounter")
        tasks.clear()
        tasks.addAll(savedInstanceState.getStringArrayList("tasks") ?: emptyList())
        updateCounterDisplay(findViewById(R.id.textCounter))
        updateTasksDisplay(findViewById<TextView>(R.id.textTasks))
    }
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)


        val textCounter = findViewById<TextView>(R.id.textCounter)
        val buttonIncrement = findViewById<Button>(R.id.buttonIncrement)
        val editTextInput = findViewById<EditText>(R.id.editTextInput)
        val buttonShow = findViewById<Button>(R.id.buttonShow)
        val textEntered = findViewById<TextView>(R.id.textEntered)
        val textTasks = findViewById<TextView>(R.id.textTasks)
        val textTotalCounter = findViewById<TextView>(R.id.textTotalCounter)
        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)
        val buttonzero = findViewById<Button>(R.id.buttonZero)
        val buttonzeroTask = findViewById<Button>(R.id.ZeroTasks)
        val buttonremlast = findViewById<Button>(R.id.ButtonRemLast)
        // Начальное значение
        updateCounterDisplay(textCounter)
        updateTotalCounterDisplay(textTotalCounter)

        buttonIncrement.setOnClickListener {
            counter++
            updateCounterDisplay(textCounter)
        }
        buttonzero.setOnClickListener{
            counter = 0
            updateCounterDisplay(textCounter)
        }
        buttonremlast.setOnClickListener {
            if (tasks.isEmpty()){
                Toast.makeText(this, "Нет задач", Toast.LENGTH_SHORT)
            } else {
                tasks.removeAt(tasks.lastIndex)
                tcounter = tasks.size
                updateTasksDisplay(textTasks)
                updateTotalCounterDisplay(textTotalCounter)
            }

        }
        buttonzeroTask.setOnClickListener {
            if (!tasks.isEmpty()){
                tasks.clear()
                tcounter = tasks.size
                updateTasksDisplay(textTasks)
                updateTotalCounterDisplay(textTotalCounter)
            } else{
                Toast.makeText(this, "Нет задач", Toast.LENGTH_SHORT).show()
            }
        }
        buttonShow.setOnClickListener {
            val inputText = editTextInput.text.toString()
            textEntered.text = getString(R.string.label_entered) + " $inputText"
        }
        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            if (task.isNotBlank()) {
                tasks.add(task)
                tcounter = tasks.size
                updateTasksDisplay(textTasks)
                updateTotalCounterDisplay(textTotalCounter)
                editTextTask.text.clear() // очищаем поле ввода


            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }
    }

    private fun updateTasksDisplay(taskview: TextView ) {

        if (tasks.isEmpty()) {
            taskview.text = getString(R.string.label_tasks)
        } else {
            taskview.text = tasks.joinToString("\n") { "• $it" }
        }
    }
    private fun updateCounterDisplay(textView: TextView) {
        textView.text = getString(R.string.counter_text, counter)
    }
    private fun updateTotalCounterDisplay(textView: TextView) {
        textView.text = getString(R.string.counter_total_text, tcounter)
    }
}

```

<br>

<img width="392" height="767" alt="image" src="https://github.com/user-attachments/assets/71d92867-6100-40de-8d26-b02192985bbc" />

Ответы на вопросы:
Как получить текст из EditText?
Почему при повороте экрана данные (счётчик, список задач) сбрасываются? Как это можно исправить?
Для чего используется joinToString? Как изменить разделитель?
В чём разница между List и MutableList?
Как очистить поле ввода после добавления задачи?

1

```kotlin

val editText = findViewById<EditText>(R.id.edit_text)
val text = editText.text.toString()

2

```


Сохранить состояние в savedInstance
Сохранение:

```kotlin

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putInt("counter", counter)
        outState.putInt("tcounter", tcounter)
        outState.putStringArrayList("tasks", ArrayList(tasks))
    }

```

Восстановление:

```kotlin

    override fun onRestoreInstanceState(savedInstanceState: Bundle) {
        super.onRestoreInstanceState(savedInstanceState)
        counter = savedInstanceState.getInt("counter")
        tcounter = savedInstanceState.getInt("tcounter")
        tasks.clear()
        tasks.addAll(savedInstanceState.getStringArrayList("tasks") ?: emptyList())
        updateCounterDisplay(findViewById(R.id.textCounter))
        updateTasksDisplay(findViewById<TextView>(R.id.textTasks))
    }

```

3
joinToString — функция для коллекций в Kotlin. Она преобразует элементы коллекции в строку, объединяя их через указанный разделитель.

val result = numbers.joinToString(separator = " | ") - разделитель передаётся через аргумент

4
list - неизменяемы список, 

mutableList - изменяемый список

<br>
5

editText.setText("")

<br>

editText.text.clear()

Вывод:

Проведена работа с сохранением состояния приложения и с изменяемым списком. 

