## Комментарии к фрагментам кода

#### **1. Функция Generator**
Функция должна генерировать числа и записывать их в канал `ch`. Для каждого числа должна быть вызвана функция `fn()`. Используйте такой алгоритм генерации:

```- N(0) = 1;
- N(i) = N(i-1) + 1
```
`Generator()` прекращает работу, когда поступил сигнал об отмене контекста `ctx`.


<details><summary><b>Подсказка 1</b></summary><p>Используйте бесконечный цикл <code>for</code> и конструкцию <code>select</code> с проверкой на <code><-ctx.Done()</code>. Перед выходом из функции закройте канал <code>ch.</code></details>

#### **2. Функция Worker()**
Эта функция обрабатывает числа. Она должна читать числа из канала `in` до тех пор, пока он не закроется. Полученное число нужно записать в канал `out` и сделать паузу на 1 миллисекунду. Если канал `in` закрылся, то нужно закрыть канал `out` и завершить работу функции.

<details><summary><b>Подсказка 2</b></summary><p> Можно использовать бесконечный цикл и оператор <code>v, ok := <-in</code>, который позволяет отследить закрытие канала.</details>

#### **3. Создание контекста**
Здесь необходимо создать контекст `ctx`, который должен самостоятельно отмениться через одну секунду.

<details><summary><b>Подсказка 3</b></summary><p> Используйте <code>context.WithTimeout()</code> и не оставляйте <code>cancel()</code> без присмотра.</details>

#### **4. Собираем числа из каналов outs**
В этом фрагменте нужно пройтись по всем каналам из слайса `outs` и для каждого из них запустить горутину с анонимной функцией. Горутина должна читать данные из переданного ей канала, до тех пор, пока он не закроется. Чтобы дождаться завершения всех этих горутин, нужно воспользоваться механизмом `WaitGroup`. Горутина должна подсчитывать обработанные числа (увеличивать соответствующий счётчик в слайсе `amounts`) передавать значения в канал `chOut`.

<details><summary><b>Подсказка 4</b></summary><p> Перед вызовом горутины не забудьте вызвать <code>wg.Add(1)</code>, а при её завершении — <code>wg.Done()</code>. Используйте анонимную функцию с двумя параметрами <code>go func(in <-chan int64, i int64){}</code>, где <code>in </code> — очередной канал из <code>outs</code>, а <code>i</code> — его индекс. Не забывайте про увеличение счётчика <code>amounts[i]++</code>.</details>

#### **5. Читаем числа из результирующего канала**
Осталось прочитать все данные из канала `chOut`. При этом нужно подсчитать в переменной `count` общее количество чисел, а в переменной `sum` — сумму всех чисел.

<details><summary><b>Подсказка 5</b></summary><p> Воспользуйтесь конструкцией <code>for ... range</code>, которая будет читать числа из канала до его закрытия.</details>

#### 6. **Исправляем потенциальное состояние гонки**
Вроде ваше программа работает и выдаёт ожидаемые результаты; но в коде есть кусок, который может привести к состоянию гонки, если программа изменится. Попробуйте найти проблемный фрагмент самостоятельно, прежде чем читать дальше.
<details><summary><b>Читать дальше</b></summary><p>
Состояние гонки в основном возникает, когда несколько горутин одновременно изменяют общие данные. Посмотрите внимательно на код программы. Предположим, кто-то решит запускать несколько горутин для генерации чисел. Это может привести к состоянию гонки: анонимная функция, которая передаётся в <code>Generator()</code>, изменяет одни и те же переменные — <code>inputSum</code> и <code>inputCount</code>. Сделайте увеличение этих переменных потокобезопасным. Вы можете использовать мьютекс или функцию <code>atomic.AddInt64()</code>.
</details>

## Проверка результатов
После добавления необходимого кода и успешной отработки, программа должна вывести в консоль примерно такие строки:
```txt
Количество чисел 4558 4558
Сумма чисел 10389961 10389961
Разбивка по каналам [913 912 913 910 910]
```

У вас значения будут другими, но числа в первой и второй строчках, которые указаны через пробел, должны совпадать. Числа слайса `amount` должны быть практически одинаковыми. Они показывают, сколько значений передалось через соответствующий канал `outs[i]`. Если через один канал прошло, например, 800 чисел, а через другой — 900, это сигнал о неполадках.
Вы можете изменить значение константы `NumOut` и сравнить результаты при `NumOut`, равной 2, 5, 10, 15. Видно, что чем больше используется горутин, тем больше чисел обрабатывается в сумме, но при этом каждая горутина обрабатывает меньше чисел. Вот пример вывода при `NumOut`, равной 15:

```txt
Количество чисел 13087 13087
Сумма чисел 85641328 85641328
Разбивка по каналам [870 873 873 873 873 873 872 872 874 872 871 872 874 871 874]
```

Если программа выдаёт ожидаемые результаты, можно отправлять её на ревью. Надеемся, что итоговое задание напомнило вам основные конструкции и инструменты работы с многопоточностью, и вы закрепили полученные знания на практике.
