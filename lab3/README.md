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

Лабораторная работа №3 
Реализация списка объектов с фильтрацией с использованием .map, .filter, .sortedBy
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

Цель работы: Изучить функциональные методы обработки коллекций в Kotlin (filter, map, sortedBy) на примере списка объектов и вывести результаты в интерфейс Android-приложения.
<br>
<br>
<br>
Листинг класса Product и MainActivity (с добавленными цепочками).
<br>

```kotlin
package com.example.msa

data class Product(
    val name: String,
    val category: String,
    val price: Double,
    val inStock: Boolean
)
```

<br>

```kotlin

package com.example.msa

import android.os.Bundle
import android.widget.TextView
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import java.sql.Date

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_main)
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }
        val products = getProducts()
        val kino = getFilms()


        // 1. Исходный список (для наглядности преобразуем в строку)
        val originalText = products.joinToString("\n") { "${it.name} – ${it.price} руб. (${if (it.inStock) "в наличии" else "нет"})" }
        findViewById<TextView>(R.id.textOriginal).text = originalText

        // 2. Фильтр: только товары в наличии
        val inStockProducts = products.filter { it.inStock }
        val inStockText = inStockProducts.joinToString("\n") { "${it.name} – ${it.price} руб." }
        findViewById<TextView>(R.id.textInStock).text = inStockText

        // 3. Цепочка: отфильтровать электронику, отсортировать по цене и получить список строк с названием и ценой
        val electronicsSorted = products
            .filter { it.category == "Электроника" && it.inStock }
            .sortedBy { it.price }
            .map { "${it.name} – ${it.price} руб." }
        val electronicsText = electronicsSorted.joinToString("\n")
        findViewById<TextView>(R.id.textSorted).text = electronicsText

        val cancelarsSorted = products
            .filter { it.category == "Канцелярия" && !it.inStock }
            .sortedByDescending { it.price }
            .map { "${it.name} – ${it.price} руб." }
        val cancelar = cancelarsSorted.joinToString("\n")
        findViewById<TextView>(R.id.cancelarSorted).text = cancelar


        var films = kino.joinToString(",\n") {"Название: ${it.name}, выпущен ${it.date.toString()}, Жанр ${it.genre}, Рейтинг IMDB - ${it.rate}"}
        var filteredfilmss = kino.filter { it.rate >=8.0 }
            .joinToString("\n") {"Название: ${it.name}, выпущен ${it.date.toString()}, Жанр ${it.genre}, Рейтинг IMDB - ${it.rate}"};
        var sortedFilms = kino.sortedBy { it.date }
            .joinToString("\n"){"Название: ${it.name}, выпущен ${it.date.toString()}, Жанр ${it.genre}, Рейтинг IMDB - ${it.rate}"}

        films = getString(R.string.Origfilm) + films;

        filteredfilmss = getString(R.string.GoodFilms) + filteredfilmss

        sortedFilms = getString(R.string.filmsYear) + sortedFilms
        findViewById<TextView>(R.id.textOriginalFilms).text = films;
        findViewById<TextView>(R.id.TextfilmsGood).text =  filteredfilmss ;
        findViewById<TextView>(R.id.textSortedfilms).text = sortedFilms;
    }
    fun getProducts(): List<Product>{
        return listOf(
            Product("Ноутбук", "Электроника", 75000.0, true),
            Product("Мышь", "Электроника", 1500.0, true),
            Product("Книга 'Котлин'", "Книги", 1200.0, false),
            Product("Флешка 64GB", "Электроника", 2000.0, true),
            Product("Блокнот", "Канцелярия", 300.0, true),
            Product("Ручка", "Канцелярия", 50.0, false),
            Product("Монитор", "Электроника", 25000.0, true)
        )
    }
        fun getFilms(): List<Film>{
            return listOf(
            Film("The Shawshank Redemption", "Drama", 9.3, Date.valueOf("1994-09-23")),
            Film("The Godfather", "Crime", 9.2, Date.valueOf("1972-03-24")),
            Film("The Dark Knight", "Action", 9.0, Date.valueOf("2008-07-18")),
            Film("Pulp Fiction", "Crime", 8.9, Date.valueOf("1994-10-14")),
            Film("Forrest Gump", "Drama", 8.8, Date.valueOf("1994-07-06")),
            Film("Inception", "Sci-Fi", 8.8, Date.valueOf("2010-07-16")),
            Film("The Matrix", "Sci-Fi", 8.7, Date.valueOf("1999-03-31")),
            Film("Goodfellas", "Crime", 8.7, Date.valueOf("1990-09-19")),
            Film("Fight Club", "Drama", 8.8, Date.valueOf("1999-10-15")),
            Film("Star Wars: Episode V - The Empire Strikes Back", "Sci-Fi", 8.7, Date.valueOf("1980-05-21")),
            Film("Avatar", "Sci-Fi/Adventure", 7.9, Date.valueOf("2009-12-18")),
            Film("Iron Man", "Action/Sci-Fi", 7.9, Date.valueOf("2008-05-02")),
            Film("The Social Network", "Biography/Drama", 7.8, Date.valueOf("2010-10-01")),
            Film("Gravity", "Sci-Fi/Thriller", 7.7, Date.valueOf("2013-10-04")),
            Film("The Hunger Games", "Action/Sci-Fi", 7.2, Date.valueOf("2012-03-23")),
            Film("The Da Vinci Code", "Mystery/Thriller", 6.6, Date.valueOf("2006-05-19"))
            )



    }

}
```
<br>


Скриншот работающего приложения (с видимыми результатами фильтрации и сортировки).
<br>
<img width="374" height="628" alt="image" src="https://github.com/user-attachments/assets/f566444e-6daf-48f4-9681-3c441da23940" />

<br>
1. Функция filter возвращает новый список, содержащий только те элементы исходной коллекции, которые удовлетворяют заданному условию (предикату). Исходная коллекция при этом не изменяется.

2. sortedBy сортирует элементы в естественном порядке возрастания (от меньшего к большему).
sortedByDescending сортирует элементы в порядке убывания (от большего к меньшему).
3. Несколько условий объединяются с помощью логических операторов && (и) и || (или) внутри предиката. Также можно комбинировать вызовы filter последовательно.
4. Функция map преобразует каждый элемент коллекции по заданному правилу и возвращает новый список результатов преобразования. Исходная коллекция не меняется.
Пример: 
```kotlin
val increasedRates = films.map { it.rate + 0.1 }
```
5. joinToString – функция для преобразования коллекции в строку с возможностью настройки разделителей, префикса, постфикса и т.д.
