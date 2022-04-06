# KPparser

Для сбора информации используются библиотеки _Selenium_ и _BeautifulSoup_.

Данные взяты с [КиноПоиска](https://www.kinopoisk.ru/top/navigator/m_act[rating]/1.1%3A/m_act[ex_rating]/1.1%3A/m_act[is_film]/on/m_act[is_mult]/on/order/rating/perpage/200/#results).

Каждая карточка фильма обрабатывается следующим кодом:

    for film in films: #сбор информации из ячейки фильма  
            name = film.find('div', class_='name').find('a').text
            country = film.find('span', class_='gray_text').text.splitlines()[1]
            country = (country + '1')[0:country.find('.') + country.find(',') + 1].strip()
            kp = film.find('div', class_='numVote').text[0:4]
            imdb = film.find('div', class_='imdb').text[6:10]
            genre = film.find('span', class_='gray_text').text.splitlines()[-2].strip()
            genre = genre[1:genre.find('.')].split(', ')

            data.append([name, country, kp, imdb] + genre)


Полученные данные выводятся в файл _Excel_
![image](https://user-images.githubusercontent.com/103055346/162056707-931380ab-0792-4112-b439-60fe5a2e3916.png)


# MovieRatingAnalysis

Целью анализа является исследование разницы оценок фильмов обычными зрителями и критиками в зависимости от жанра. В качестве данных используются оценки пользователей сайта КиноПоиск и критиков сайта IMDb.
