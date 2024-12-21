## Задача №1. Deadlock

### Легенда

У вас есть кусок кода, который почему-то не работает:

```kotlin
package ru.netology.deadlock

import kotlin.concurrent.thread

fun main() {
    val resourceA = Any()
    val resourceB = Any()

    val consumerA = Consumer("A")
    val consumerB = Consumer("B")

    val t1 = thread {
        consumerA.lockFirstAndTrySecond(resourceA, resourceB)
    }
    val t2 = thread {
        consumerB.lockFirstAndTrySecond(resourceB, resourceA)
    }

    t1.join()
    t2.join()

    println("main successfully finished")
}

class Consumer(private val name: String) {
    fun lockFirstAndTrySecond(first: Any, second: Any) {
        synchronized(first) {
            println("$name locked first, sleep and wait for second")
            Thread.sleep(1000)
            lockSecond(second)
        }
    }

    fun lockSecond(second: Any) {
        synchronized(second) {
            println("$name locked second")
        }
    }
}
```
### Вопрос

«*По какой причине не завершается работа функции `main`?»*

### Ответ

При выполнении кода происходит взаимная блокировка, которая возникает, когда два или более потока блокируют ресурсы, которые другие потоки пытаются заблокировать в обратной последовательности.

Synchronized устанавливает объект, монитор которого используется для защиты блока кода от параллельного исполнения разными потоками.
Поток перед входом в блок synchronized проверяет монитор объекта: если он свободен, то захватывает его, если нет — ждёт, пока монитор освободится.

В коде два потока t1 и t2 запускаются параллельно и пытаются захватить два ресурса resourceA и resourceB в разных последовательностях.
Поскольку t1 пытается сначала захватить resourceA, а затем resourceB, а t2 делает наоборот, сначала захватывая resourceB, а затем resourceA, есть вероятность, что они заблокируют друг друга и не смогут завершить выполнение.

Для успешного завершения метода main нам надо убрать блок synchronized с объекта либо в функции lockFirstAndTrySecond либо в функции lockSecond.


