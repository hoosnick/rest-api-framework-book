# Mavzu 5: QuerySet. Mantiqiy shartlar va agregat funksiyalar

## Reja:

1. [Bilim](#1-bilim)
   - [1.1 Terminlar](#11-terminlar)
   - [1.2 O'qish uchun materiallar](#12-oqish-uchun-materiallar)
2. [Amaliyot. O'qituvchi](#2-amaliyot-oqituvchi)
3. [Amaliyot. O'quvchi](#3-amaliyot-oquvchi)

## 1. Bilim

### 1.1 Terminlar

```

```

## 2. Amaliyot. O'qituvchi

**Reja:**

- [2.1 Shartlar bilan ishlash]
  - [2.1.1 Mantiqiy OR]
  - [2.1.1 Mantiqiy AND]
- [2.2 Agregat funksiyalar]
  - [2.2.1 Count]
  - [2.2.2 Avg]
  - [2.2.3 Min]
  - [2.2.4 Max]
  - [2.2.5 Sum]

![](images/queryset/img_6.png)

### 2.1 Shartlar bilan ishlash

#### 2.1.1 Mantiqiy OR

1. Loyiha nomida IT yoki .uz so'zlari bo'lgan loyihalar ro'yxatini chiqaring

<p>1-usul</p>

```text
>>> Project.objects.filter(title__icontains='it') | Project.objects.filter(title__icontains='.uz')

<QuerySet [<Project: IT Academy online ta'lim>, <Project: ePark.uz>, <Project: Dolina capital>]>
```

<p>2-usul</p>

```text
>>> from django.db.models import Q
>>> Project.objects.filter(Q(title__icontains='.uz') and Q(title__icontains='it'))

<QuerySet [<Project: IT Academy online ta'lim>, <Project: Dolina capital>]>
```

2. Foydalanuvchi ro'yxatdan o'tgan oy aprel, may yoki dekabr bo'lgan foydalanuvchilarni chiqaring

Avval hammasini ko'rib olamiz

```text
>>> for pr in Profile.objects.all(): print(pr, pr.created)

Asror 2022-01-06
Murod 2022-02-06
Husniddin 2022-03-08
Diyor 2022-04-08
Akbar 2021-05-08
Bekzod 2022-06-08
Sherzod 2022-07-08
Eshmat 2022-07-08
Toshmat 2022-12-08
Jasur 2022-12-08
```

<p>Endi filtrlaymiz:</p>

<p>1-usul</p>

```text
>>> for pr in Profile.objects.filter(created__month__in=[4,5,12]): print(pr, pr.created)

Diyor 2022-04-08
Akbar 2021-05-08
Toshmat 2022-12-08
Jasur 2022-12-08
```

<p>2-usul</p>

```text
>>> for pr in Profile.objects.filter(Q(created__month=4) | Q(created__month=5) | Q(created__month=12)): print(pr, pr.created)

Diyor 2022-04-08
Akbar 2021-05-08
Toshmat 2022-12-08
Jasur 2022-12-08
```

#### 2.1.2 Mantiqiy AND

3. github v instagram bor bo'lgan dasturchilari chiqaring

```text
>>> for pr in Profile.objects.filter(Q(social_github__isnull=False) & Q(social_instagram__isnull=False)): print( pr.social_instagram,pr.social_github, pr.user.username)

https://www.instagram.com/husnidd1n_17/ https://17husniddin.github.io/Potfolio/index.html Husniddin
```

4. 2022 yil yanvardan aprelgacha ro'yxatdan o'tganlarni chiqaring

```text
>>> for pr in Profile.objects.filter(Q(created__year=2022) & Q(created__month__range=[1,4])): print(pr.user.username,pr.created)

Asror 2022-01-06
Murod 2022-02-06
Husniddin 2022-03-08
Diyor 2022-04-08
```

### 2.2 Agregat funksiyalar

Agregat funksiyalar avval ma'lum guruhga tegishli ma'lumotni yig'ib oladi, so'ng uni qayta ishlaydi
<br>[To'liqlor ma'lumot](https://docs.djangoproject.com/en/4.0/topics/db/aggregation/)

#### 2.2.1 Count

Ikki hil usulda ishlatish mumkin:

- hamma qatorlar soni
- ma'lum guruhga tegishli qiymatlar soni

Avg o'rtacha qiymatni topish uchun ishlatiladi

5. Hamma loyihalar sonini chiqaring

```text
>>> Profile.objects.count()

10
```

6. Nomida it so'zi uchragan loyihalar sonini chiqaring

```text
>>> Project.objects.filter(title__icontains='it').count()

2
```

7. Asror bildirgan fikrlar sonini aniqlang

<br>
Avval filtrlab olamiz

```text
>>> Review.objects.filter(user__user__username='Asror')

<QuerySet [<Review: Zo'r loyiha bo'libdi>, <Review: CSS, HTML da qilinganmi>]>
```

Endi sonini chiqaramiz

```text
>>> Review.objects.filter(user__user__username='Asror').count()

2
```

##### Endi ma'lum guruhga tegishli qiymatlar sonini aniqlashni ko'ramiz

8. Har bir foydalanuvchi nechtadan loyihasi borligini chiqaring

```text
>>> from django.db.models import Count
>>> for pr in Profile.objects.annotate(total_projects=Count('project')):
...     print(pr, pr.total_projects)

Asror 1
Murod 1
Husniddin 1
Diyor 3
Jasur 0
Akbar 0
Bekzod 0
Sherzod 0
Eshmat 0
Toshmat 0
```

#### 2.2.2 Avg

Avg - o'rtacha qiymatni topib, dict qaytaradi

9. Loyihaga berilgan o'rtacha ovozlar sonini aniqlang

```text
>>> from django.db.models import Avg
>>> Project.objects.aggregate(Avg('vote_count'))

{'vote_count__avg': 138.33333333333334}
```

yoki dict dagi kalitni o'zimiz bersak ham bo'ladi

```text
>>> from django.db.models import Avg
>>> Project.objects.aggregate(avarage=Avg('vote_count'))

{'avarage': 138.33333333333334}
```

#### 2.2.3 Min

Min - eng kichigini(son, sana) topib, dict qaytaradi.

10. Loyihaga berilgan eng kichik ovozni chiqaring

```text
>>> from django.db.models import Min
>>> Project.objects.aggregate(min=Min('vote_count'))

{'min': 30}
```

#### 2.2.4 Max

11. Loyihaga berilgan eng katta ovozni chiqaring

```text
>>> from django.db.models import Max
>>> Project.objects.aggregate(max=Max('vote_count'))

{'max': 300}
```

#### 2.2.5 Sum

12. Loyihaga berilgan ovozlar yig'indisini chiqaring

```text
>>> from django.db.models import Sum
>>> Project.objects.aggregate(sum=Sum('vote_count'))

{'sum': 830}
```

13. Loyihaga berilgan ovozlar sonini min, max, o'rtacha va yig'indisini chiqaring

```text
>>> Project.objects.aggregate(sum=Sum('vote_count'), avg=Avg('vote_count'), max=Max('vote_count'), min=Min('vote_count'))

{'sum': 830, 'avg': 138.33333333333334, 'max': 300, 'min': 30}
```

##### Birga ko'p bog'lanishda agregat funskiyalar

![Sxema](images/queryset/img_6.png)

14. Hamma foydalanuvchining hamma malakasi sonini chiqaring

```text
>>> Profile.objects.aggregate(Count('user_skills'))

{'user_skills__count': 11}
```

15. Har bir foydalanuvchining nechtadan malakasi bor?

```text
>>> for pr in Profile.objects.annotate(soni = Count('user_skills')):
...    print(pr, pr.soni)

Asror 3
Murod 2
Husniddin 3
Diyor 3
Jasur 0
Akbar 0
Bekzod 0
Sherzod 0
Eshmat 0
Toshmat 0
```

16. Har bir tegda nechtadan loyiha qilingan

```text
>>> for tag in Tag.objects.annotate(soni = Count('project_tag')):
...   print(tag, tag.soni)

Python 2
React 3
Django 1
Django Rest Framework 1
Javascript 5
CSS,HTML 3
test 0
```

Xulosa, masalan, Pythonda ikkita loyiha qilingan ekan

17. Har bir tegda nechtadan loyiha qilinganligini tartiblab chiqaring

```text
>>> for tag in Tag.objects.annotate(soni = Count('project_tag')):
...   print(tag, tag.soni)

Python 2
React 3
Django 1
Django Rest Framework 1
Javascript 5
CSS,HTML 3
test 0
```

## 3. Amaliyot. O'quvchi

1. 10 ta yoki 20 ovoz olgan loyihalarni chiqaring
2. Yuzgacha yoki 1000dan ortiq ovoz olgan loyihalarni chiqaring
3. 2022 yil sentyabrdan dekabrgacha ro'yxatdan o'tgan foydalanuvchilar ro'yxati
4. 2021 yil yanvardan aprelgacha va 2022 yil sentyabrdan dekabrgacha ro'yxatdan o'tgan foydalanuvchilar ro'yxati
5. Hamma foydalanuvchilar sonini chiqaring
6. Hamma habarlar sonini chiqaring
7. Hamma fikrlar sonini chiqaring
8. Asrorga yuborilgan habarlar soni
9. Asror yuborgan habarlar soni
10. 100 dan ortiq ovoz olgan loyihalar soni
11. Har bir foydalanuvchi nechtadan habar jo'natgan
12. Har bir foydalanuvchi nechtadan habar qabul qilgan
13. Review modelidagi o'rtacha value ni toping
14. Loyihaga berilgan eng kichik ovozni chiqaring
15. Loyihaga berilgan eng katta ovozni chiqaring
16. Har bir loyihada nechta teg (soha, dasturlash tili) bor?
17. Har bir loyihaga nechtadan fikr bildirilgan
18. Har bir foydalanuvchiga nechtadan habar jo'natilgan
