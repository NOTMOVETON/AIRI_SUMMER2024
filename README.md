# Репозиторий проекта Со-дизайн робота гибридной локомоции летней школы AIRI

[Отчет](https://docs.google.com/document/d/1tdW2XGwlyqLLGHqRc42NOWkJK84U0Exm723ULfpPi38/edit?usp=sharing)
[Презентация работы](https://docs.google.com/presentation/d/1jtNIakRO1FqaYClxj6sjHEq2X0mII-rKI4AufXIBggU/edit?usp=sharing)

### Изначальная идея
Изначально идея заключалась в реализации задачи со-дизайна робота гибридной локомоции (конструкция адаптируется под управление и наоборот), но с учетом недостаточного для полного выполнения задачи времени, было принято решение реализовать задачу езды с заданной скоростью.


### Шаги реализации
В процессе работы над проектом, мы придерживались следующих шагов:
- Изучение вариантов создания симуляции (MuJoCo, MJX). Остановились на MuJoCo, так как вникать в JAX за очень короткий срок оказалось довольно неприятно.
- Выбор библиотеки с реализацией алгоритмов обучения с подкреплением. (Пробовали использовать самописные - долго и реализация у каждого отличается, что сводит совместную работу на нет). Остановились на stable-baselines3
- Создание среды с агентом, совместимой с выбранной библиотекой, поддержка параллелизации(несколько агентов обучаются одновременно)
- Тестирование разных алгоритмов, с разными модификациями среды (DQN, SAC, PPO). Остановились на PPO, так как позволял работать как с дискретными, так и с непрерывными действиями, а также показал себя лучше чем остальные методы (скорее всего у них просто были неправильно настроены гиперпараметры, но времени у нас было не очень много)
- Тестирование разных вариаций задач. Начали с простого - удержание равновесия и позиции, далее усложняли задачу - удержание желаемой высоты, езда с заданной, езда с заданной линейной и угловой скоростями, езда к точке. В течении каждого из шагов меняли среду, добавляли новое, тестировали, и так пока не заработает.
- Reward-engineering. Много экспериментировали с наградами, вводили разные награды с разным соотношением, тестировали нормирование как наград, так и показаний с робота.


### Результаты
В результате выполнения проекта было обучено огромное множество обученных агентов, большинство из них не могли справиться с поставленной задачей, но некоторые показали относительно рабочий результат. 

https://github.com/user-attachments/assets/e38d4ec8-0903-4f6c-8c7a-77d67349bb0e

Видео с примером работы лучшего полученного агента

Данный агент был обучен для устойчивого движения с заданной линейной и угловой скоростями. Для реализации использовался алгоритм Proximal Policy Optimization (PPO) из библиотеки stable-baselines3.

#### Архитектура модели

Агент обучен на основе полносвязной нейронной сети с архитектурой $\{in_{dim}, 64, 64, out_{dim}\}$, одинаковой для актёра и критика. Это стандартная архитектура из stable-baselines3, которая используется по умолчанию для PPO.
#### Функции наград

В процессе обучения были определены три компонента награды:
1. Скорость: Пропорционально уменьшение награды по мере роста отклонения текущей скорости агента от желаемой.
   $ R_v = -|v - v_{target}| $

2. Положение: Поощрение за нахождение в положении, близком к вручную заданному угловому состоянию $\theta_{target}$, обеспечивающему равновесие.
   $ R_{\theta} = -\sum_{i=1}^{n} |\theta_i - \theta_{target_i}| $

3. Штраф за отклонение: Снижение награды пропорционально увеличению расстояния от стартовой позиции $(x_{start}, y_{start})$.
   $ R_p = -\sqrt{(x - x_{start})^2 + (y - y_{start})^2} $

Каждый эпизод обучения использует случайную выборку линейной и угловой скоростей из множества значений $\{-0.5, -0.4, ..., 0.5\}$ с шагом 0.1, которые добавляются в вектор наблюдений агента.

## Модификация среды

Для сокращения времени сходимости и улучшения управляемости непрерывные действия были заменены дискретными:
- Управление ногами: движения по $\pm15^{\circ}$ с шагом $1^{\circ}$ для каждого из сочленений.
- Колёса: управление с шагом $\pm1$, соответствующее движению вперёд или назад на небольшой скорости.
Это позволило уменьшить количество эпизодов, затрачиваемое агентом для исследования возможностей собственного тела.

## Результаты

Агент достиг относительно устойчивого поведения за 750 000 шагов (15-20 минут обучения на CPU с 8 параллельными средами). Агент способен двигаться со скоростями в правильном направлении, но движение происходит с некоторыми нарушениями плавности и отклонением от начальной позиции, что может быть связано с не оптимальной постановкой функций наград, недостаточным временем обучения, либо неправильно подобранными гиперпараметрами.

## Заключение

Метод PPO продемонстрировал положительные результаты для данной задачи, но требует дальнейшей оптимизации для улучшения качества движений и повышения точности удержания заданной траектории. Помимо этого следует рассмотреть другие архитектуры актера и критика, а также использовать другие подходы для обучения, например использование привелегированных данных для обучения критика, что очень важно на этапе обучения.



