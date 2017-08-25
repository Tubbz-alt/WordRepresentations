# Сравнение нескольких способов представления слов для построения языковой модели с использованием градиентного бустинга и нейросетей.

## Решаемая задача

Этот экспериментальный бенчмарк сравнивает разные способы представления слов
в упрощенной задаче построения так называемой language model (https://en.wikipedia.org/wiki/Language_model).

Language Model - это способ вычислить вероятность заданного фрагмента текста (цепочки слов).
В NLP ранее широко использовались различные варианты построения LM на базе подсчета
употребления N-грамм и сглаживания, чтобы скомпенсировать разряженность для N-грамм
большой арности. К примеру, в [NLTK](http://www.nltk.org/) есть реализация [метода Kneser-Ney](http://www.nltk.org/_modules/nltk/probability.html).
Также существует большое количество разных реализаций, отличающихся техническими деталями касательно
хранения больших объемов N-граммной информации и скоростью обработки текста, например [https://github.com/kpu/kenlm](https://github.com/kpu/kenlm).

Мы немного упростим нашу Langage Model, поставив задачу в таком виде. Есть N-грамма заранее выбранной длины (см. константу NGRAM_ORDER 
в модуле [DatasetVectorizers](https://github.com/Koziev/WordRepresentations/blob/master/PyModels/DatasetVectorizers.py)).
Если она получена из текстового корпуса (путь к зазипованному
utf-8 файлу прошит в методе _get_corpus_path класса BaseVectorizer), то считаем, что
это валидное сочетание слов и целевое значение y=1. Если же N-грамма получена случайной
заменой одного из слов и такая цепочка не встречается в корпусе, то целевое значение y=0.

Недопустимые N-граммы генерируются в ходе анализа корпуса в том же количестве, что и валидные.
Получается сбалансированный датасет, что облегчает задачу. А отсутствие необходимости ручной разметки
позволяет легко "играться" таким параметром, как количество записей в обучающем датасете.

Таким образом, решается бинарная классификационная задача. Классификатором будет реализация
градиентного бустинга XGBoost и нейросеть, реализованная на Keras, а также несколько
других вариантов на разных языках программирования и DL фреймворках.

Объектом исследования является влияние способа представления слов во входной матрице X
на точность классификации.

## Варианты представления слов

Для XGBoost проверяются следующие варианты:

* **w2v** - используем word2vec, склеивая векторы слов в один длинный вектор  

* **sdr** - sparse distributed representations слов, получаемые факторизацией матрицы word2vector модели  

* **random_bitvector** - каждому слову приписывается случайный бинарный вектор фиксированной длины с заданной пропорцией 0/1  

* **bc** - в качестве репрезентаций используются векторы, созданные в результате работы brown clustering (см. описание https://en.wikipedia.org/wiki/Brown_clustering и реализацию https://github.com/percyliang/brown-cluster)  

* **chars** - каждое слово кодируется как цепочка из 1-hot репрезентаций символов  

* **hashing_trick** - используется hashing trick для кодирования слов ограниченным числом битов индекса   (см. описание https://en.wikipedia.org/wiki/Feature_hashing и реализацию https://radimrehurek.com/gensim/corpora/hashdictionary.html)  

Для нейросетки используется слой Embedding, поэтому внешнее представление слов - целочисленные
индексы, а фактическое внутреннее - векторы встраивания, настраиваемые в ходе оптимизации
нейросети.

## Модули и решатели

**Запускаемые программы на Python:**  
[PyModels/wr_xgboost.py](https://github.com/Koziev/WordRepresentations/blob/master/PyModels/wr_xgboost.py) - решатель на базе XGBoost (Python)  
[PyModels/wr_catboost.py](https://github.com/Koziev/WordRepresentations/blob/master/PyModels/wr_catboost.py) - решатель на базе CatBoost по индексам слов, использующий возможность указать индексы категориальных признаков в датасете, чтобы бустер самостоятельно учел их при тренировке (Python)  
[PyModels/wr_keras.py](https://github.com/Koziev/WordRepresentations/blob/master/PyModels/wr_keras.py) - решатель на базе feed forward нейросетки, реализованной на Keras (Python)  
[PyModels/wr_lasagne.py](https://github.com/Koziev/WordRepresentations/blob/master/PyModels/wr_lasagne.py) - решатель на базе feed forward нейросетки, реализованной на Lasagne (Theano, Python)  
[PyModels/wr_nolearn.py](https://github.com/Koziev/WordRepresentations/blob/master/PyModels/wr_nolearn.py) - решатель на базе feed forward нейросетки, реализованной на nolearn+Lasagne (Theano, Python)  

**Компилируемые программы на C#:**  
[CSharpModels/WithAccordNet/Program.cs](https://github.com/Koziev/WordRepresentations/blob/master/CSharpModels/WithAccordNet/Program.cs) - решатель на базе feed forward сетки Accord.NET (C#, проект для VS 2015)  

**Компилируемые программы на C++:**  
[CXXModels/TinyDNN_Model/TinyDNN_Model.cpp](https://github.com/Koziev/WordRepresentations/blob/master/CXXModels/TinyDNN_Model/TinyDNN_Model.cpp) - решатель на базе MLP, реализованного средствами библиотеки tiny-dnn (C++, проект для VS 2015)  
[CXXModels/Singa_Model/alexnet.cc](https://github.com/Koziev/WordRepresentations/blob/master/CXXModels/Singa_Model/alexnet.cc) - решатель на базе нейросетки, реализованной средствами Apache.SINGA (C++, проект для VS 2015)  
[CXXModels/OpenNN_Model/main.cpp](https://github.com/Koziev/WordRepresentations/blob/master/CXXModels/OpenNN_Model/main.cpp) - решатель на базе нейросетки, реализованной средствами OpenNN (C++, проект для VS 2015)  

**Внутренние классы и инструменты:**  
PyModels/DatasetVectorizers.py - векторизаторы датасета и фабрика для удобного выбора векторизатора по его условному названию  
PyModels/store_dataset_file.py - генерация датасета и его сохранение для C# и C++ моделей  


## Нейросетевые архитектуры

Для нейросеточного решения на базе Keras реализовано 2 архитектуры.
**MLP** - простая feed forward нейросетка с полносвязанными слоями.
**ConvNet** - сетка, в которой используются [сверточные слои](https://keras.io/layers/convolutional/).

Переключение архитектур выполняется настроечным параметром NET_ARCH в файле wr_keras.py.

Можно заметить, что сверточный вариант зачастую дает более точное решение, но соответствующие
модели имеют больший variance, попросту говоря - итоговая точность сильно скачет для
нескольких моделей с одинаковыми параметрами.


## Формат корпуса

Все варианты бенчмарка используют тектовый файл в кодировке utf-8 для получения
списков N-грамм. Предполагается, что разбивка текста на слова и приведение к нижнему
регистру выполнены заранее сторонним кодом. Поэтому скрипты читают строки из этого файла,
разбивают их на слова по пробелам.

Для удобства воспроизведения экспериментов я поместил в папку data сжатую выдержку
из большого корпуса размером 100 тысяч строк. Этого достаточно для повторения всех
замеров. Чтобы не перегружать репозиторий большим файлом, он зазипован. Метод BaseVectorizer._load_ngrams
сам распаковывает его содержимое на лету, поэтому никаких ручных манипуляций делать не надо.


## Грамматический словарь

В варианте векторизации w2v_tags к w2v-векторам слов добавляются морфологические признаки.
Эти признаки для каждого слова берутся из упакованного файла word2tags_bin.zip в подкаталоге data,
который получен конвертацией моего грамматического словаря (http://solarix.ru/sql-dictionary-sdk.shtml).
Результат конвертации тянет на 165 Мб, что многовато для git репозитория. Поэтому я 
зазиповал получившимйся файл грамматического словаря, а соответствующий класс
векторизации датасета сам распаковывает его на лету во время работы.

Обратите внимание, что из-за омонимии грамматических форм многие слова имеют более
одного варианта набора тегов. Например, слово 'мишки' может быть формой единственного
или множественного числа соответственно в родительном или именительном падеже. При формировании
наборов тегов для слов я объединяю теги омонимов.


## Получение sparse distributed representation для слов

Используется двухэтапный процесс.
Сначала получаем w2v векторы слов. Небольшой скрипт на питоне выгружает векторы
в текстовый файл в csv формате.

Затем с помощью утилиты https://github.com/mfaruqui/sparse-coding выполняется команда:  

./nonneg.o input_vectors.txt 16 0.2 1e-5 4 out_vecs.txt

Генерируется файл out_vecs.txt, содержащий SDR слов с указанными параметрами: размер=16*длина вектора w2v, заполнение=0.2
Этот файл грузится классом SDR_Vectorizer.


## Baseline  

В качестве проверки используем запоминание N-грамм из тренировочного набора и
поиск среди запомненных N-грамм для проверочного набора. Небольшой размер тренировочного
набора приводит к тому, что на проверочном наборе модель ведет себя просто как рандомный
угадыватель, давая точность 0.50.


## Текущие результаты

Далее представлены результаты для датасета, содержащего 1,000,000 триграмм.

Для решателя на базе XGBoost (wr_xgboost.py):

w2v_tags (word2vector + morph tags) ==> 0.81  
w2v (word2vector dim=32) ==> 0.80  
sdr (sparse distributed representations, длина 512, 20% единиц) ==> 0.78
bc (brown clustering) ==> 0.70  
chars (char one-hot encoding) ==> 0.71  
random_bitvector (random bit vectors, доля единиц 16%) ==> 0.696  
hashing_trick (hashing trick with 32,000 slots) ==> 0.64  


Для решателя на базе Keras feed forward neural net (wr_keras.py):  

NB_SAMPLES=1,000,000  NGRAM_ORDER=2  w2v_tags  acc=0.8340  
NB_SAMPLES=2,000,000  NGRAM_ORDER=2  w2v_tags  acc=0.8405  
NB_SAMPLES=3,000,000  NGRAM_ORDER=2  w2v_tags  acc=0.8474  
NB_SAMPLES=5,000,000  NGRAM_ORDER=2  w2v_tags  acc=0.8511  
NB_SAMPLES=10,000,000 NGRAM_ORDER=2  w2v_tags  acc=0.8561  

NB_SAMPLES=1,000,000  NGRAM_ORDER=2  word_indeces  acc=0.7452  
NB_SAMPLES=2,000,000  NGRAM_ORDER=2  word_indeces  acc=0.7619  
NB_SAMPLES=3,000,000  NGRAM_ORDER=2  word_indeces  acc=0.7735  
NB_SAMPLES=5,000,000  NGRAM_ORDER=2  word_indeces  acc=0.7835  
NB_SAMPLES=6,000,000  NGRAM_ORDER=2  word_indeces  acc=0.7849  
NB_SAMPLES=10,000,000 NGRAM_ORDER=2  word_indeces  acc=0.8103  

NB_SAMPLES=1,000,000  NGRAM_ORDER=2  w2v  acc=0.7954  
NB_SAMPLES=2,000,000  NGRAM_ORDER=2  w2v  acc=0.8010  
NB_SAMPLES=3,000,000  NGRAM_ORDER=2  w2v  acc=0.8050  
NB_SAMPLES=5,000,000  NGRAM_ORDER=2  w2v  acc=0.8065  
NB_SAMPLES=6,000,000  NGRAM_ORDER=2  w2v  acc=0.8086  
NB_SAMPLES=10,000,000 NGRAM_ORDER=2  w2v  acc=0.8132  


NB_SAMPLES=1,000,000  NGRAM_ORDER=3  w2v_tags  acc=0.8528  
NB_SAMPLES=2,000,000  NGRAM_ORDER=3  w2v_tags  acc=0.8585  
NB_SAMPLES=3,000,000  NGRAM_ORDER=3  w2v_tags  acc=0.8662  
NB_SAMPLES=5,000,000  NGRAM_ORDER=3  w2v_tags  acc=0.8738  
NB_SAMPLES=6,000,000  NGRAM_ORDER=3  w2v_tags  acc=0.8760  
NB_SAMPLES=8,000,000  NGRAM_ORDER=3  w2v_tags  acc=0.8774  

                      
NB_SAMPLES=1,000,000  NGRAM_ORDER=3  word_indeces  acc=0.7719  
NB_SAMPLES=2,000,000  NGRAM_ORDER=3  word_indeces  acc=0.7807  
NB_SAMPLES=3,000,000  NGRAM_ORDER=3  word_indeces  acc=0.7875  
NB_SAMPLES=5,000,000  NGRAM_ORDER=3  word_indeces  acc=0.8250  
NB_SAMPLES=6,000,000  NGRAM_ORDER=3  word_indeces  acc=0.8266  
NB_SAMPLES=8,000,000  NGRAM_ORDER=3  word_indeces  acc=0.8328  

NB_SAMPLES=1,000,000  NGRAM_ORDER=3  w2v  acc=0.8116  
NB_SAMPLES=2,000,000  NGRAM_ORDER=3  w2v  acc=0.8140  
NB_SAMPLES=3,000,000  NGRAM_ORDER=3  w2v  acc=0.8191  
NB_SAMPLES=5,000,000  NGRAM_ORDER=3  w2v  acc=0.8280  
NB_SAMPLES=6,000,000  NGRAM_ORDER=3  w2v  acc=0.8275  
NB_SAMPLES=8,000,000  NGRAM_ORDER=3  w2v  acc=0.8289  

NB_SAMPLES=1,000,000  NGRAM_ORDER=4  w2v_tags  acc=0.8376  
NB_SAMPLES=2,000,000  NGRAM_ORDER=4  w2v_tags  acc=0.8575  
NB_SAMPLES=3,000,000  NGRAM_ORDER=4  w2v_tags  acc=0.8701  
NB_SAMPLES=5,000,000  NGRAM_ORDER=4  w2v_tags  acc=0.8789  
NB_SAMPLES=6,000,000  NGRAM_ORDER=4  w2v_tags  acc=0.8751  
NB_SAMPLES=8,000,000  NGRAM_ORDER=4  w2v_tags  acc=0.8805  



Для решателя на базе Lasagne MLP:

accuracy=0.71  

Для решателя на базе nolearn+lasagne MLP:

accuracy=0.64

Для решателя на базе tiny-dnn MLP:

accuracy=0.58

Для решателя на базе Accord.NET feed forward neural net:

word2vector ==> 0.68  

Для решателя на базе CatBoost по категориальным признакам "индекс слова":

accuracy=0.50  

Для решателя на базе Apache.SINGA MLP:  

accuracy=0.74  

Baseline решение - запоминание N-грамм из тренировочного датасета

accuracy=0.50


## Дополнительные подробности:  
https://habrahabr.ru/post/335838/  
http://kelijah.livejournal.com/217608.html  
