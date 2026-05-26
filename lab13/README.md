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

Лабораторная работа №13
Создание простого API клиента. Запрос списка постов с jsonplaceholder.typicode.com
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
Листинг post
<br>

```kotlin
package com.example.lab132.Model

data class Post(
    val id: Int,
    val title: String,
    val body: String
)
```

<br>
листинг
<br>

```kotlin


import com.example.lab132.Model.Post
import retrofit2.http.GET
import retrofit2.http.Query

interface ApiService {
    @GET("posts")
    suspend fun getPosts(
        @Query("_page") page: Int,
        @Query("_limit") limit: Int
    ): List<Post>
}
```

<br>

листинг RetrofitClient.kt

<br>

```kotlin
package com.example.lab132.api

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitClient {
    private const val BASE_URL = "https://jsonplaceholder.typicode.com/"

    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    val apiService: ApiService = retrofit.create(ApiService::class.java)
}
```

<br>

листинг PostsRepository.kt

<br>

```kotlin
import com.example.lab132.Model.Post
import com.example.lab132.api.RetrofitClient
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

class PostsRepo {
    private val apiService = RetrofitClient.apiService

    suspend fun getPosts(page: Int, limit: Int = 20): List<Post> =
        withContext(Dispatchers.IO) {
            apiService.getPosts(page, limit)
        }
}
```

<br>
Листинг postviewmodel
<br>

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.lab132.Model.Post
import com.example.lab132.repository.PostsRepo
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch

sealed class PostsUiState {
    object Loading : PostsUiState()
    data class Success(val posts: List<Post>) : PostsUiState()
    data class Error(val message: String) : PostsUiState()
}

class PostsViewModel : ViewModel() {
    private val repository = PostsRepo()
    private val allPosts = mutableListOf<Post>()
    private var currentPage = 0
    private var isLoadingMore = false
    private var isLastPage = false

    private val _uiState = MutableStateFlow<PostsUiState>(PostsUiState.Loading)
    val uiState: StateFlow<PostsUiState> = _uiState.asStateFlow()
    fun isLoadingMore(): Boolean = isLoadingMore
    init {
        loadFirstPage()
    }

    private fun loadFirstPage() {
        viewModelScope.launch {
            _uiState.value = PostsUiState.Loading
            try {
                currentPage = 1
                val posts = repository.getPosts(page = currentPage, limit = 20)
                allPosts.clear()
                allPosts.addAll(posts)
                isLastPage = posts.size < 20
                _uiState.value = PostsUiState.Success(allPosts.toList())
            } catch (e: Exception) {
                _uiState.value = PostsUiState.Error(e.message ?: "Ошибка")
            }
        }
    }

    fun loadNextPage() {
        if (isLoadingMore || isLastPage) return
        isLoadingMore = true
        viewModelScope.launch {
            try {
                delay(1000)
                currentPage++
                val newPosts = repository.getPosts(page = currentPage, limit = 20)
                if (newPosts.isEmpty()) {
                    isLastPage = true
                } else {
                    allPosts.addAll(newPosts)
                    _uiState.value = PostsUiState.Success(allPosts.toList())
                    isLastPage = newPosts.size < 20
                }
            } catch (e: Exception) {
                currentPage--
            } finally {
                isLoadingMore = false
            }
        }
    }

    fun refresh() {
        loadFirstPage()
    }
}
```

<br>
Листниг PostsAdapter
<br>

```kotlin


import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.example.lab132.Model.Post
import com.example.lab132.R


class PostsAdapter :
    RecyclerView.Adapter<PostsAdapter.PostViewHolder>() {
    private var posts = emptyList<Post>()


    class PostViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textPostId: TextView = itemView.findViewById(R.id.textPostId)
        val textPostBody: TextView = itemView.findViewById(R.id.textPostBody)
        val textPostTitle: TextView = itemView.findViewById(R.id.textPostTitle)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PostViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_post, parent, false)
        val hldr = PostViewHolder(view)

        return hldr
    }

    override fun onBindViewHolder(
        holder: PostViewHolder,
        position: Int
    ) {
        val post = posts[position]
        holder.textPostId.text = post.id.toString()
        holder.textPostBody.text = post.body
        holder.textPostTitle.text = post.title
    }


    fun submitList(newPosts: List<Post>) {
        posts = newPosts
        notifyDataSetChanged()
    }
    override fun getItemCount() = posts.size


}
```

<br>
листинг MainActivity

<br>

```kotlin
import android.os.Bundle
import android.widget.Button
import android.widget.ProgressBar
import android.widget.TextView
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.lab132.ViewModel.PostsUiState
import com.example.lab132.ViewModel.PostsViewModel
import com.example.lab132.adapters.PostsAdapter

import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val viewModel: PostsViewModel by viewModels()
    private lateinit var adapter: PostsAdapter
    private lateinit var recyclerView: RecyclerView
    private lateinit var progressBar: ProgressBar
    private lateinit var textError: TextView
    private lateinit var buttonRefresh: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        recyclerView = findViewById(R.id.recyclerViewPosts)
        progressBar = findViewById(R.id.progressBar)
        textError = findViewById(R.id.textError)
        buttonRefresh = findViewById(R.id.buttonRefresh)

        setupRecyclerView()
        observeUiState()

        buttonRefresh.setOnClickListener {
            viewModel.refresh()
        }
    }

    private fun setupRecyclerView() {
        adapter = PostsAdapter()
        val layoutManager = LinearLayoutManager(this)
        recyclerView.layoutManager = layoutManager
        recyclerView.adapter = adapter

        recyclerView.addOnScrollListener(object : RecyclerView.OnScrollListener() {
            override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
                super.onScrolled(recyclerView, dx, dy)
                val totalItemCount = layoutManager.itemCount
                val lastVisibleItem = layoutManager.findLastVisibleItemPosition()
                // Загружаем следующую страницу, когда видны последние 3 элемента (можно и 1)
                if (totalItemCount <= lastVisibleItem + 3 && !viewModel.isLoadingMore()) {
                    viewModel.loadNextPage()
                }
            }
        })
    }
    private fun observeUiState() {
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    when (state) {
                        is PostsUiState.Loading -> showLoading()
                        is PostsUiState.Success -> showPosts(state.posts)
                        is PostsUiState.Error -> showError(state.message)
                    }
                }
            }
        }
    }

    private fun showLoading() {
        recyclerView.visibility = android.view.View.GONE
        progressBar.visibility = android.view.View.VISIBLE
        textError.visibility = android.view.View.GONE
    }

    private fun showPosts(posts: List<com.example.lab132.Model.Post>) {
        recyclerView.visibility = android.view.View.VISIBLE
        progressBar.visibility = android.view.View.GONE
        textError.visibility = android.view.View.GONE
        adapter.submitList(posts)
    }

    private fun showError(message: String) {
        recyclerView.visibility = android.view.View.GONE
        progressBar.visibility = android.view.View.GONE
        textError.visibility = android.view.View.VISIBLE
        textError.text = "Ошибка: $message"
    }
}
```

<br>
листинг activity_main.xml
<br>

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:id="@+id/buttonRefresh"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Обновить"/>

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerViewPosts"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:visibility="gone"/>

        <ProgressBar
            android:id="@+id/progressBar"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:visibility="gone"/>

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
ЛИстинг post_item
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
    app:cardElevation="4dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <TextView
            android:id="@+id/textPostId"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="ID: "
            android:textStyle="bold"
            android:textSize="14sp"/>

        <TextView
            android:id="@+id/textPostTitle"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Title"
            android:textSize="18sp"
            android:textStyle="bold"
            android:layout_marginTop="4dp"/>

        <TextView
            android:id="@+id/textPostBody"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Body"
            android:textSize="14sp"
            android:layout_marginTop="8dp"/>

    </LinearLayout>
</androidx.cardview.widget.CardView>
```

<br>

<img width="261" height="487" alt="image" src="https://github.com/user-attachments/assets/60526e19-b829-45a4-8b22-1f146604d8ac" />

<img width="261" height="501" alt="image" src="https://github.com/user-attachments/assets/4e671895-01e7-47ea-b342-c12cbb7e1c3c" />

<img width="246" height="483" alt="image" src="https://github.com/user-attachments/assets/ff47b0ea-d4d9-4a9a-b327-1a3d9044e812" />

<br>

## 1 Для чего используется библиотека Retrofit? Какие аннотации вы знаете?
Retrofit — это типобезопасный HTTP-клиент для Android и Java, созданный компанией Square. Его основное назначение — упростить взаимодействие с REST API, избавляя разработчика от низкоуровневой работы с HttpURLConnection или HttpClient.

Зачем нужен:

Декларативное описание API: достаточно создать интерфейс и расставить аннотации.

Автоматическое преобразование JSON (или XML) в Kotlin-объекты с помощью подключаемых конвертеров (например, GsonConverterFactory).

Аннотации:
Http аннотации: @GET, @POST, @PUT, @DELETE, @PATCH, @HEAD, @OPTIONS, @HTTP
@Query - параметр GET

<br>

## 2 Почему сетевые запросы нельзя выполнять в главном потоке?
Главный поток (UI-поток) отвечает за отрисовку интерфейса, обработку нажатий, анимации и другие пользовательские взаимодействия. Если в этом же потоке выполнить сетевой запрос (или другую длительную операцию), поток будет заблокирован на всё время ожидания ответа.

<br>

## 3 Что такое suspend функция и как она работает с корутинами?
suspend-функция — это функция, которая может быть приостановлена в процессе выполнения без блокировки потока, на котором она запущена. Она является фундаментом асинхронного программирования на корутинах в Kotlin

<br>

## 4 Для чего нужен Dispatchers.IO?
Dispatchers.IO — это один из встроенных диспетчеров корутин в Kotlin. Он специально предназначен для операций ввода-вывода: сетевых запросов, чтения/записи файлов, работы с базой данных.

<br>

## 5 Как обрабатывать ошибки при сетевых запросах?
Конкретный тип исключений зависит от используемого HTTP-клиента, но общая схема одинакова:

Обернуть вызов в try-catch.

Ловить ошибки сети: IOException (нет соединения, таймаут).

Обрабатывать ошибки ответа сервера: не‑успешные HTTP-статусы. Некоторые клиенты (например, OkHttp напрямую) не выбрасывают исключение на 4xx/5xx — нужно проверять response.isSuccessful и парсить код. Другие (Ktor, Retrofit с suspend) выбрасывают исключение, которое можно поймать.

Отдельно ловить ошибки парсинга данных.
## 6 Что такое JSONPlaceholder и для чего он используется?
Это бесплатный фейковый REST API, отдающий тестовые JSON-данные (посты, комментарии, пользователи, фото и т.п.).
Не требует регистрации, всегда доступен по адресу https://jsonplaceholder.typicode.com.
Используется для обучения и отладки HTTP-клиентов, создания прототипов, демонстрации работы с сетью — идеально, когда реального бэкенда ещё нет.

## Вывод
- В ходе работы освоено создание API-клиента на Retrofit с корутинами для асинхронных запросов
- Реализована загрузка списка постов с JSONPlaceholder
- Добавлена постраничная подгрузка данных при прокрутке.

