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

Графики получены для  
lr1-2.ipynb [colab](https://drive.google.com/file/d/1WC7Kj7vAP50uCvAhsZko9JZ1IdiROllK/view?usp=sharing);  
CPU - Intel(R) Xeon(R) CPU @ 2.00GHz (2 ядра, 4 потока);  
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
