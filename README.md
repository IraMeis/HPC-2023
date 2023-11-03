# HPC-2023  
## Matrix Multiplication

Вначале появилась реализация на питоне (lr1/lr1-1.ipynb).
Там обычный ijk-алгоритм: создается двумерная сетка,
нить с индексом (i, j) вычисляет C(i, j).

...Но матрицу 
1000х1000 последовательный питон считал ~7 минут, мне надоело ждать, и я
~~достала свою старую лабу по параллельному программированию~~
написала вариант реализации на плюсах (lr1/lr1-2.ipynb). 
Тут вместо изобретания
велосипедов используется cublas'овское апи
(для массива double - cublasDgemm, за контекст отвечает cublasHandle_t).

Графики для:  
[lr1-2.ipynb](https://drive.google.com/file/d/1WC7Kj7vAP50uCvAhsZko9JZ1IdiROllK/view?usp=sharing);  
CPU - Intel(R) Xeon(R) CPU @ 2.00GHz (1 ядро, 2 потока);  
GPU - Tesla T4.

![График времени](https://github.com/IraMeis/HPC-2023/blob/main/lr1/tm.png)  
![График ускорения](https://github.com/IraMeis/HPC-2023/blob/main/lr1/ac.png)  

Еще немного про вариант с питоном (и 25 мин джобу).  
Запускала у себя урезанную версию compare (только cpu),
с матрицами 100, 200 и дальше до 2000 с шагом 200,
хватило интереса на 25 мин, и вот результаты на моей машине 
Intel(R) Core(TM) i7-8565U CPU @ 1.80GHz,
4 ядра, 8 потоков (но по факту все труъ-последовательное,
питон работает только в 1 потоке).

| 100       | 200       | 400       | 600       | 800       | 1000      | 1200     |
|-----------|-----------|-----------|-----------|-----------|-----------|----------|
| 0.376113s | 2.994133s | 24.92170s | 94.80045  | 213.1737s | 427.5922s | 721.610s |

## Vector Sum

Еще одна лаба в стиле "сделай и на питоне и на плюсах", 
но тут реализация совпадает. Редукция:
нити блока добаляют свой элемент из общего вектора в 
буфер разделяемой памяти,
после синхронизации 0 нить суммирует все что находится 
в буфере и атомарно запиывает в результирующую 
глобальную переменную.


Графики для:  
[lr2.ipynb](https://drive.google.com/file/d/1J0OMkKVSYwoSxUw92XqMsxzpi2AIpkbU/view?usp=sharing);  
CPU - Intel(R) Xeon(R) CPU @ 2.00GHz (1 ядро, 2 потока);  
GPU - Tesla T4.

**C++**  
На интервале N 1e3 - 5e6 получается ускорение до 20 - 23 раз.  
![График времени](https://github.com/IraMeis/HPC-2023/blob/main/lr2/cpp-tm.png)  
![График ускорения](https://github.com/IraMeis/HPC-2023/blob/main/lr2/cpp-ac.png)  


**Python**  
Для питона сравнивались gpu, cpu/обычный for и cpu/np.sum на 1е3 - 1е8.
Numpy быстрее всех (ожидаемо).

Тут интересен скорее график падения ускорения нампая к gpu.  
![График ускорения](https://github.com/IraMeis/HPC-2023/blob/main/lr2/np2gpu.png)  
Еще наблюдение - питон-gpu по времени 
выдает стабильно ок. 0.15с (?? питоновские куда-библиотеки,
переводящие в плюсы, в данном случае не 
могут нашлепать более оптимизированного кода ??), 
поэтому ускрорение возникает 
только после 1е6, до этой размерности 
последовательное быстрее. 


Ускорение вообще (gpu vs cpu/обычный for)  
![График ускорения](https://github.com/IraMeis/HPC-2023/blob/main/lr2/py-ac1.png)  
Ускорение появляется (gpu vs cpu/обычный for)  
![График ускорения](https://github.com/IraMeis/HPC-2023/blob/main/lr2/py-ac2.png)  

**C++ vs Python**   
Разница по времени между последовательными и 
куда-версиями на плюсах и питоне.  
![График времени](https://github.com/IraMeis/HPC-2023/blob/main/lr2/dif-tm.png)  
![График времени](https://github.com/IraMeis/HPC-2023/blob/main/lr2/dif-cuda.png)  
Ускорение плюсов к питону.  
![График ускорения](https://github.com/IraMeis/HPC-2023/blob/main/lr2/acc-seq.png)   
![График ускорения](https://github.com/IraMeis/HPC-2023/blob/main/lr2/acc-cuda.png)   

## Bilateral Filtering
Python. Параллелится вычисление нового значения
для каждого пикселя (одна нить - один пиксель):
cuda.grid(2) выдает уникальные 2д координаты
треда в гриде (маппинг структуры 2д грида в 
2д матрицу), проверяется что координата
не выходит за границы картинки, далее 
расчет заначения по формуле.  

Значения получены для:  
[lr3.ipynb](https://drive.google.com/file/d/1oh6_QK-XlyKD8WV9t36N8rZaDkJThhyt/view?usp=sharing);  
CPU - Intel(R) Xeon(R) CPU @ 2.00GHz (1 ядро, 2 потока);  
GPU - Tesla T4.

| dim       | cpu       | gpu        | acceleration |
|:----------|:----------|:-----------|:-------------|
| 224x224   | 18.980194 | 0.01045328 | 1815.7165    |
| 512x512   | 96.438258 | 0.03741945 | 2577.2223    |
| 1000x1000 | 375.08709 | 0.13271145 | 2826.3354    |

Питон медленный...

Котэ с блюром и без  
![кот2](https://github.com/IraMeis/HPC-2023/blob/main/lr3/cat_bl.png)
![кот1](https://github.com/IraMeis/HPC-2023/blob/main/lr3/cat_no_bl.png)  

## Substrings Search  
Python. Параллелится основная итерация (для
(n, k) производится декремент рабочей матрицы при 
совпадении с символом буфера). Здесь аналогично
предыдущей лабе используется cuda.grid(3)
(теперь 3д сетка) выдающая уникальные 3д координаты
треда в гриде, x, y - координаты рабочей матрицы,
z-координата интерпретируется как 
индекс символа подстроки. От описанного в задании
реализация отличается только тем, что подстроки 
фиксированной длины (нампай не очень работает с 
массивами переменной длины, а делать все подстроки
по длине максимальной + забивать спецсимволами 
в подстроках поменьше лишние символы - не очень
красивый вариант, поэтому сделано на фиксированных).

Матрица сама по себе содержит данные о позиции
найденной подстроки, но анализ массива, где
по индексу подстроки лежит позиция подстроки,
быстрее. Поэтому был сделан еще один результирующий массив.

**Показатели качества:**  
- совпадение рабочих матриц
- совпадение результирующих массивов индексов по наличию подстроки (cpu_found ~= gpu_found) 
- совпадение результирующих массивов индексов по позиции подстроки (cpu_found == gpu_found)

По первым двум показателям последовательный и 
параллельный варианты совпадают всегда, по третьему -
с переменным успехом.  
Выяснилось, что если сделать  

`cuda.atomic.add(found, y, - found[y] + x - k 
if R[y, x - k] == 0 and (found[y] > x - k 
or found[y] == -1) else 0)`  

то if будет не особо атомарный и при больших 
размерностях потоки
все равно воруют друг у друга значения.  

Если сделать попытку "умным" пербором вытащить
первый индекс вхождения подстроки, то на больших 
размерностях по невыясненным причинам этот
варианттоже ломается.
```
cuda.syncthreads()  
if k == 0 and R[y, x] == 0  
  for i in range(x - 1, -1, -1):  
    if R[y, i] == 0:  
      return  
  cuda.syncthreads()  
  found[y] = x  
```    
Первый вариант работает в среднем лучше,
результаты далее представлены для него.

Значения получены для:  
[lr4.ipynb](https://drive.google.com/file/d/1V-hQiuFKxdiALc_bDr7eX4yfP4YX9ZLj/view?usp=sharing);  
CPU - Intel(R) Xeon(R) CPU @ 2.00GHz (1 ядро, 2 потока);  
GPU - Tesla T4,  
длина подстроки  2.

|index|gpu time|cpu time|acceleration|cpu\_R == gpu\_R|cpu\_found ~= gpu\_found|cpu\_found == gpu\_found|
|---|---|---|---|---|---|---|
|N\_10\_\_H\_5|0\.0014541759490966798|5\.62667846679687e-05|0\.038693243897288454|true|true|true|
|N\_100\_\_H\_50|0\.0018134399652481078|0\.004182338714599609|2\.306301170564198|true|true|true|
|N\_1000\_\_H\_500|0\.003163104057312012|0\.3898618221282959|123\.25292341460252|true|true|true|
|N\_5000\_\_H\_2500|0\.0241014404296875|12\.084508895874023|501\.4019361676265|true|true|true|
|N\_10000\_\_H\_5000|0\.0906362533569336|43\.908902168273926|484\.4518671282316|true|true|false|
