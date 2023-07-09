# Answers

_Данный проект содержит ответы на вопросы домашних заданий_

## __ДЗ__ https://github.com/netology-code/andin-homeworks/tree/master/09_supervision


### Вопросы: Cancellation

#### Вопрос №1

Отработает ли в этом коде строка `<--`? Поясните, почему да или нет.
__ОТВЕТ:__ Не отработает, 
т.к. ей не хватит времени запуститься до отмены родительской корутины, а после отмены родителя никакие дети не запускаются.

```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
    }
    delay(100) 
    job.cancelAndJoin()
}
```

#### Вопрос №2

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.
__ОТВЕТ:__ Не отработает, 
т.к. delay(500) уже началась и должна сначала завершиться, а когда delay завершится, то сначала корутина проверится, можно ли ей продолжать, а ее как раз уже отменили, и она не продолжится.

```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        val child = launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
        delay(100) 
        child.cancel()
    }
    delay(100)
    job.join()
}
```

### Вопросы: Exception Handling

#### Вопрос №1

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.
__ОТВЕТ:__ Нет.
Потому что для перехвата исключения try должен быть внутри launch, а тут наоборот - launch внутри try.
В лекции было показано, что именно в этом случае catch не срабатывает.
И что в try/catch нужно заключать не саму корутину, а потенциально опасную операцию внутри нее.

```kotlin
fun main() {
    with(CoroutineScope(EmptyCoroutineContext)) {
        try {
            launch {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```

#### Вопрос №2

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.
__ОТВЕТ:__ Да.
Исключение дочерней корутины попадает в блок try/catch родительской корутины в результате перехвата внутри coroutineScope - специальной функции для организации такого перехвата, и отмеченная строка выполняется.


```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```

#### Вопрос №3

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.
__ОТВЕТ:__ Да.
Исключение дочерней корутины попадает в блок try/catch родительской корутины посредством supervisorScope - специальной области, которая пробрасывает исключение в родительскую корутину, в результате чего отмеченная строка выполняется.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```

#### Вопрос №4

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.
__ОТВЕТ:__ Нет. 
В этой корутине успеет запуститься только delay(500).
Исключение второй дочерней корутины пробрасывается в родительскую корутину, а все дочерние корутины в этой родительской отменяются. 
Поскольку delay(500) завершится через достаточное время после начала отмены родительской корутины, то строка со стрелкой выполниться не успеет.


```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
    Thread.sleep(1000)
}
```

#### Вопрос №5

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.
__ОТВЕТ:__ Да. 
Потому что supervisorScope не отменяет родительскую корутину при ошибке в дочерней.
Соответственно, вторая дочерняя продолжает выполняться, пока в ней не возникнет ее собственная ошибка.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```

#### Вопрос №6

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.
__ОТВЕТ:__ Нет. 
Ошибка в родительской корутине вызовет отмену всех дочерних.
Соответственно, в дочерних успеет выполниться не более, чем delay.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```

#### Вопрос №7

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.
__ОТВЕТ:__ Нет. 
Потому что ошибка происходит в корутине, родительской для двух дочерних.
SupervisorJob помогает не отменять родительскую, но продолжить выполнение дочерних при ошибке в родительской он не может.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext + SupervisorJob()).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```



## __ДЗ__ https://github.com/netology-code/and2-homeworks/tree/master/07_crud

|__Вопрос__|__Ответ__|
|---|---|
|Выберите любой пост для редактирования. Удалите его, пока он находится в форме для редактирования, при помощи меню с тремя точками. После сохранения пост должен исчезнуть. <br/> За счёт какой конструкции — проверки, функции и т. п. — пост не добавляется заново, как при обычном нажатии кнопки добавления поста? Чтобы ответить на этот вопрос, изучите код нашей реализации.|Пост не добавится вновь за счет того, что  ветка _else_ никогда не выполнится (ведь этот пост к моменту сохранения уже отсутствует в списке): <br/> `if (it.id != post.id) it else it.copy(content = post.content)` <br/>Также не будет конфликта, если мы лайкнем пост, находящийся в режиме редактирования, поскольку та же самая ветка _else_ выполнится с использованием id и content, без перезаписывания любых других полей (а в it уже будет новое значение лайка)|



## __ДЗ__ https://github.com/netology-code/and2-homeworks/tree/master/04_events#задача-parent-child

|__Вопрос__|__Ответ__|
|---|---|
|Какой из обработчиков сработал при клике на кнопку Like?|собственный обработчик кнопки Like|
|Сработал ли обработчик на binding.root при клике на кнопку с тремя точками?|нет|
|Сработал ли обработчик на binding.root при клике на текст?|нет|
|Сработал ли обработчик на binding.root при клике на аватар до установки на avatar собственного обработчика?|да|
|Сработал ли обработчик на binding.root при клике на аватар после установки на avatar собственного обработчика?|нет, вместо этого сработал собственный обработчик|
|Попробуйте выявить закономерность: когда срабатывает обработчик на контейнере, а когда нет.|Предположительно обработчик binding НЕ СРАБАТЫВАЕТ в двух следующих случаях: <BR>1) При НАЛИЧИИ собственного обработчика. <BR>2) Когда элемент считается ПОТЕНЦИАЛЬНО активным: если это кнопка (даже кнопка без заданного обработчика) или если для текста установлен атрибут autoLink. <BR>В остальных случаях обработчик binding срабатывает.|

