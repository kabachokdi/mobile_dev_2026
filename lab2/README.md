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

Лабораторная работа №2 
 Написание консольных утилит на Kotlin внутри Android проекта. Расчеты, работа со строками. Подготовка классов данных для будущего приложения 
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


Листинги созданных классов данных и функций-утилит.
```kotlin
package com.example.mfa.expack

data class Book(
    val title: String,
    val author: String,
    val year: Int,
    val price: Double
)

```
```kotlin
package com.example.mfa.expack

data class student(
    val name: String,
    val sname: String,
    val group: String,
    val middle: Double
)

```
```kotlin
package com.example.mfa.expack

// Проверка, что строка похожа на email (содержит @ и .)
fun String.isValidEmail(): Boolean {
    return this.contains("@") && this.contains(".")
}

// Форматирование автора: "Толстой Л.Н."
fun formatAuthorName(fullName: String): String {
    val parts = fullName.split(" ").filter { it.isNotBlank() }
    return when (parts.size) {
        1 -> parts[0]  // только фамилия
        2 -> "${parts[0]} ${parts[1].first()}."  // фамилия и инициал
        3 -> "${parts[0]} ${parts[1].first()}.${parts[2].first()}."  // фамилия и два инициала
        else -> fullName
    }
}

// Применение скидки к цене книги
fun applyDiscount(price: Double, discountPercent: Double): Double {
    require(discountPercent in 0.0..100.0) { "Скидка должна быть от 0 до 100" }
    return price * (1 - discountPercent / 100)

}
fun studentRank(student: student): String
{
    val rank: String = when(student.middle) {
        in 2.0..3.0 -> "двоечник"
        in 3.0+0.00000001..4.0 -> "троечник"
        in 4.0+0.00000001..5.0-0.00000001 -> "хорошист"
        5.0 -> "отличник"
        else -> "непонятный"
    }
    return "${student.name} ${student.sname} из группы ${student.group} - $rank"
}
```
Листинги юнит-тестов.
```kotlin
class StringUtilsTest {

    @Test
    fun emailValidation_correct() {
        assertTrue("test@example.com".isValidEmail())
        assertTrue("user.name@domain.co".isValidEmail())
    }

    @Test
    fun emailValidation_incorrect() {

        assertFalse("test@example".isValidEmail())
        assertFalse("".isValidEmail())
    }
    @Test
    fun studentRank_correction()
    {
        assertEquals("Иванов Иван из группы 331 - троечник", studentRank(student("Иванов", "Иван", "331", 3.3)))
        assertEquals("Иванов Иван из группы 331 - непонятный", studentRank(student("Иванов", "Иван", "331", 1.0)))
        assertEquals("Иванов Иван из группы 331 - отличник", studentRank(student("Иванов", "Иван", "331", 5.0)))

    }

    @Test
    fun formatAuthorName_fullName() {
        assertEquals("Толстой Л.Н.", formatAuthorName("Толстой Лев Николаевич"))
        assertEquals("Пушкин А.С.", formatAuthorName("Пушкин Александр Сергеевич"))
    }

    @Test
    fun formatAuthorName_twoParts() {
        assertEquals("Толстой Л.", formatAuthorName("Толстой Лев"))
        assertEquals("Пушкин А.", formatAuthorName("Пушкин Александр"))
    }

    @Test
    fun formatAuthorName_onePart() {
        assertEquals("Толстой", formatAuthorName("Толстой"))
    }

    @Test
    fun applyDiscount_normal() {
        assertEquals(90.0, applyDiscount(100.0, 10.0), 0.001)
        assertEquals(75.0, applyDiscount(150.0, 50.0), 0.001)
    }

    @Test
    fun applyDiscount_zero() {
        assertEquals(100.0, applyDiscount(100.0, 0.0), 0.001)
    }

    @Test(expected = IllegalArgumentException::class)
    fun applyDiscount_invalidLow() {
        applyDiscount(100.0, -5.0)
    }

    @Test(expected = IllegalArgumentException::class)
    fun applyDiscount_invalidHigh() {
        applyDiscount(100.0, 110.0)
    }
}
```
Скриншот выполненных тестов<br>
<img width="1449" height="79" alt="image" src="https://github.com/user-attachments/assets/3797068e-d809-4b5e-8c0f-af616450f8af" />
<br>Скриншот ui<br> 
<br><img width="258" height="476" alt="image" src="https://github.com/user-attachments/assets/e5c96960-2455-48a6-87ce-e5aec932b45b" /><br>

Контрольные вопросы 
1. Data class - структура данных, служит для повышения читаемости кода, облегчения работы с данными
2. Обычная функция - функция которая обьявленна внутри класса, пакета. Функция расширения - функция описанная вне класса, но которая вызывается от имени обьекта класса, самый ходовой пример - расширение класса String, так как невозможно создать класс-наследник String (тк он строка является sealed-классом), бывает удобно ввести новую функцию для обработки строки
3. <br> <img width="589" height="502" alt="image" src="https://github.com/user-attachments/assets/2619d743-c3c0-4f21-a9e6-5fefa71a3f89" /><br>
4. Так как все действия с числами с плавающей точкой происходят приближённо (с точностью до n-го знака), при сравнении достаточно близких чисел может возникать ошибочное срабатываение. Для этого используется assertEquals, который позволяет "отсечь" часть числа<br>
5.<br><img width="254" height="172" alt="image" src="https://github.com/user-attachments/assets/2ffdf7db-d7a1-42ae-9a25-8367f00ad315" /><br>
Выводы
Освоены базовые элементы обработки данных в kotlin, основы применения Unit-тестов
