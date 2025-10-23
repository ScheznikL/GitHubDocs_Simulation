## Комп'ютерні системи імітаційного моделювання
## СПм-22-11, **Іванов Іван Іванович**
### Лабораторна робота №**2**. Редагування імітаційних моделей у середовищі NetLogo

<br>

### Варіант 0, модель у середовищі NetLogo:
[Traffic Basic](http://www.netlogoweb.org/launch#http://www.netlogoweb.org/assets/modelslib/Sample%20Models/Social%20Science/Traffic%20Basic.nlogo)

<br>

### Внесені зміни у вихідну логіку моделі, за варіантом:

**Виправлення розміщення активних агентів** на ігровому полі при ініціалізації моделі - спочатку машини могли розміщуватися на тієї ж самої ділянці дороги.  
Замість
<pre>
set xcor abs random-xcor
</pre>
у процедурі setup-cars використовується виклик окремої нової процедури розміщення агентів:
<pre>
to set-freeposition
  set xcor random-pxcor
  if any? other turtles-here [ set-freeposition ]
end
</pre>
Стара процедура separate-cars була видалена.  
Додавання функції для отримання випадкового числа в заданому діапазоні для частого подальшого використання в інших змінах логіки моделі
<pre>
to-report get-random-float [ low rand ] 
  report low + random-float rand
end
</pre>
Наприклад, використовувалася при встановленні початкового значення швидкості активних агентів:
<pre>
set speed get-random-float 0.1 speed-limit / 10
</pre>
та при встановленні обмеження максимальної швідкості для поточного агенту:
<pre>
set speed-limit get-random-float 1.1 0.9
</pre>

**Встановлення ліміту максимальної швидкості для кожної машини здійснюється індивідуально**, а не однаково для всіх машин:
<pre>
;; set speed-limit 1
set speed-limit get-random-float 1.1 0.9
</pre>
Використовується кольорова диференціація машин залежно від їхньої швидкісного ліміту:
<pre>
  ;; set label speed-limit
  if(speed-limit > 1.2) [
      set color green
  ]
   if(speed-limit > 1.5) [
     set color red
  ]

  ;; ask sample-car [ set color red ]
</pre>
Сині найповільніші, зелені швидше, червоні найшвидші.  
Забарвлення обраної для відстеження машини скасовано.

**Зміна логіки гальмування та набору швидкості** залежно від наявності перешкоди перед машиною:
<pre>
  let car-ahead-1 one-of turtles-on patch-ahead 1
    let car-ahead-2 one-of turtles-on patch-ahead 2
    ifelse car-ahead-1 = nobody and car-ahead-2 = nobody
      [ speed-up-car ] ;; otherwise, speed up
      [ slow-down-car car-ahead-1 car-ahead-2 ]
    ;; don't slow down below speed minimum or speed up beyond speed limit
    if speed < speed-min [ set speed speed-min ]
    if speed > speed-limit [ set speed speed-limit ]
    fd speed
</pre>
Для цього також були внесені зміни до процедури slow-down-car, яка спрацьовує при гальмуванні:
<pre>
to slow-down-car [ car-ahead-1 car-ahead-2 ] ;; turtle procedure
  if (car-ahead-1 != nobody) 
  [
      let speed-car-ahead-1 [ speed ] of car-ahead-1
      ;; slow down so you are driving more slowly than the car ahead of you
      set speed speed-car-ahead-1 - deceleration
      stop
  ]
  ]  
  if (car-ahead-2 != nobody) [
    let speed-car-ahead-2 [ speed ] of car-ahead-2
    let speed-difference-ahead-2 abs speed - speed-car-ahead-2
    set speed speed - speed-difference-ahead-2 / 2
  ]
end
</pre>

<br>

### Внесені зміни у вихідну логіку моделі, на власний розсуд:

**Додано ймовірність безвідповідальності водія, який, при нагоді, їздитиме узбіччям**.
Імовірність встановлюється користувачем через інтерфейс середовища моделювання (слайдер для bad-driver-probability) та використовується при додаванні машин на полі:
<pre>
globals [
  sample-car
  speed-min
  car-roadside-amount
  bad-driver-probability
]

;; виїзд на узбіччя здійснюється, якщо перед машиною є інша машина і ще одна, водій "поганий", знаходиться на дорозі (бо з узбіччя з'їжджати далі нікуди)
if (car-ahead-1 != nobody) [
    ;; перевіряємо, чи буде на даному тіку водій "недисциплінованим"
    let rand random 100
    ifelse(rand < bad-driver-probability) [
      ;; якщо недисциплінований, і при цьому є додаткова перешкода перед машинойї перед нами, то переміщуємось на узбіччя
      if(car-ahead-2 != nobody) [
        ;; перевірка, чи знаходимось ми зараз на дорозі і чи вільне узбіччя
        if (is-on-road) and (roadside-free-check) [
          set heading 180
          fd 1
          set heading 90
          ;; збільшуємо лічильник "узбічників"
          set car-roadside-amount car-roadside-amount + 1
        ]
      ]
    ]
  ]
</pre>
Лічильник "поганих водіїв" виводитиметься користувачеві.

Перед виїздом на узбіччя, воне проглядається водієм, чи вільно там, за допомогою функції is-roadside-free:
<pre>
to-report roadside-free-check
  set heading 180
  let car-roadside one-of turtles-on patch-ahead 1
  set heading 90
  ;ifelse(car-roadside = nobody) [
    report car-roadside = nobody
  ;;]
  ;;[
  ;;  report false
  ;;]
end
</pre>

Пеервірка, чи знаходимось на дорозі:
<pre>
to-report is-on-road
    report pycor = 0
end
</pre>

Додано узбіччя, дорога зведена до повноційної односмугової:
<pre>
to setup-road ;; patch procedure
  if pycor < 1 and pycor > -2 [ set pcolor yellow ]
  if pycor < 1 and pycor > -1 [ set pcolor white ]
end
</pre>

На кожному ході виконується перевірка, чи знаходиться машина на дорозі. Якщо машина їде узбіччям, то намагатиметься повернутися на дорогу:
<pre>
  if(not is-on-road) [
      if (road-main-free-check) [
        set heading 0
        fd 1
        set heading 90
        set car-roadside-amount car-roadside-amount - 1
      ]
    ]
</pre>
Функція перевірки, чи вільна дорога поблизу машини, щоб можна було повернутися з узбіччя:
<pre>
to-report road-main-free-check
  set heading 0
  let car-road one-of turtles-on patch-ahead 1
  set heading 90
  ;ifelse(car-roadside = nobody) [
    report car-road = nobody
  ;;]
  ;;[
  ;;  report false
  ;;]
end
</pre>

![Скріншот моделі в процесі симуляції](example-model.png)

Фінальний код моделі та її інтерфейс доступні за [посиланням](example-model.nlogo). *// якщо вносили зміни до інтерфейсу середовища моделювання - то експорт потрібен у форматі nlogo, як тут. Інакше, якщо змінювався лише код логіки моделі, достатньо викласти лише його, як [тут](example-model-code.html),якщо експортовано з десктопної версії NetLogo, або окремим текстовим файлом, шляхом копіпасту з веб-версії*.
<br>

## Обчислювальні експерименти
*// тут повинен бути наведений опис одного експерименту, за аналогією з першої л/р.* 
### 1. Вплив дисциплінованості водіів на середню швидкість переміщення
