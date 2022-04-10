# KPparser

Для сбора информации используются библиотеки _Selenium_ и _BeautifulSoup_.

Данные взяты с [КиноПоиска](https://www.kinopoisk.ru/top/navigator/m_act[rating]/1.1%3A/m_act[ex_rating]/1.1%3A/m_act[is_film]/on/m_act[is_mult]/on/order/rating/perpage/200/#results).

Каждая карточка фильма обрабатывается следующим кодом:
```
    for film in films: #сбор информации из ячейки фильма  
            name = film.find('div', class_='name').find('a').text
            country = film.find('span', class_='gray_text').text.splitlines()[1]
            country = (country + '1')[0:country.find('.') + country.find(',') + 1].strip()
            kp = film.find('div', class_='numVote').text[0:4]
            imdb = film.find('div', class_='imdb').text[6:10]
            genre = film.find('span', class_='gray_text').text.splitlines()[-2].strip()
            genre = genre[1:genre.find('.')].split(', ')

            data.append([name, country, kp, imdb] + genre)
```

Полученные данные выводятся в файл _Excel_
![image](https://user-images.githubusercontent.com/103055346/162056707-931380ab-0792-4112-b439-60fe5a2e3916.png)


# MovieRatingAnalysis

Целью анализа является исследование разницы оценок фильмов обычными зрителями и критиками в зависимости от жанра. В качестве данных используются оценки пользователей сайта КиноПоиск и критиков сайта IMDb.

Используемые библиотеки:
```{python}
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import statsmodels.api as sm
import scipy.stats as stats
import scikit_posthocs as sp
```
Удаляются фильмы с жанром, общее количество которых меньше 100
```
cleardf = list(data['Жанр_1'].value_counts()[data['Жанр_1'].value_counts()<100].index)
data.query(f'Жанр_1 not in {cleardf}', inplace = True)
```
и создается таблица с новой колонкой разности рейтингов 
```
datarating = data.get(['Название', 'Жанр_1', 'Страна'])
datarating['rating'] = data['КП'] - data['IMDB']
```
![image](https://user-images.githubusercontent.com/103055346/162632829-f3ca650b-61fd-4669-ae36-1859636decbf.png)

Генеральная совокупность имеет ассиметричное ненормальное распределение: 

![image](https://user-images.githubusercontent.com/103055346/162634788-b922e36e-5519-4628-ae56-d2990e41182b.png)
```
stats.normaltest(datarating['rating'])
NormaltestResult(statistic=3285.1006553280413, pvalue=0.0)
```
Критерий Краскела — Уоллиса для проверки равности распределения:
```
groups = []
for group in list(drg.groups):
    groups.append(list(drg.get_group(group)['rating'].round(3)))   #создание пожанрового списка рейтингов 
stats.kruskal(*groups) #непараметрический тест на равность распределений
KruskalResult(statistic=1639.646012402479, pvalue=0.0)
```

![image](https://user-images.githubusercontent.com/103055346/162635985-bd5e1cca-7651-487c-925a-d9dd758ba651.png)

Тест Данна для попарного сравнения:
```
dp = sp.posthoc_dunn(datarating, val_col = 'rating', group_col = 'Жанр_1', p_adjust = 'hommel') 
dp.apply(lambda x: x > 0.5).style.highlight_max(color = 'yellowgreen').highlight_min(color='coral') 
```

![image](https://user-images.githubusercontent.com/103055346/162637087-7d9047c8-24d3-4243-8746-022f45d394c1.png)
