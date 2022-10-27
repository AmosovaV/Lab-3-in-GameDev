# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил(а):
- Амосова Варвара Ивановна
- РИ-210940
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | # | 60 |
| Задание 2 | # | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.

## Задача работы
Создать ML-агент и попробовать потренировать нейросеть, задача которой будет заключаться в управлении шаром. Задача шара заключается в том, чтобы оставаясь на плоскости находить кубик, смещающийся в заданном случайном диапазоне координат.

## Задание 1
### Пошагово выполнить каждый пункт раздела "ход работы" с описанием и примерами реализации задач
Ход работы:
- Для начала необходимо создать пустой проект на Unity, куда далее добавляем ML Agent. 

![2022-10-25_22-34-50](https://user-images.githubusercontent.com/114309754/197881533-381eb999-d7d1-4508-a8ee-b1c1538fb242.png)

- Проверка правильности добавления.

![2022-10-25_22-35-38](https://user-images.githubusercontent.com/114309754/197881613-360d7821-e8f8-479e-9a62-1dfed0cf993e.png)

- Далее пишем серию команд для создания и активации нового ML-агента, а также для скачивания необходимых библиотек.

![Снимок экрана (25)](https://user-images.githubusercontent.com/114309754/197882593-e8fd867f-19fc-4627-983d-7cbe94423fac.png)
![Снимок экрана (26)](https://user-images.githubusercontent.com/114309754/197882589-f199b4eb-e8c1-4d83-90d6-dbfd029017a8.png)
![Снимок экрана (27)](https://user-images.githubusercontent.com/114309754/197882586-c0090983-913d-4990-a569-077634557b63.png)
![Снимок экрана (28)](https://user-images.githubusercontent.com/114309754/197882582-32c4242d-65d8-4fe7-b16f-3025cb59cf82.png)
![Снимок экрана (30)](https://user-images.githubusercontent.com/114309754/197882580-dc08bfac-f3ac-4c2a-ad61-3ea28249bd39.png)

- Создаем на сцене плоскость, куб и сферу. 

![2022-10-26_01-38-21](https://user-images.githubusercontent.com/114309754/197882995-2bae7b23-072f-403d-8487-52e6125c8101.png)
![2022-10-26_00-11-36](https://user-images.githubusercontent.com/114309754/197884014-d8febbb6-a6e7-474f-bd4c-fab59370259a.png)
![2022-10-26_00-11-24](https://user-images.githubusercontent.com/114309754/197884023-ea90b593-213b-4a17-8264-63ec76df0422.png)

- Создаем C# скрипт-файл и подключаем его к сфере:

![2022-10-26_01-39-12](https://user-images.githubusercontent.com/114309754/197883371-0c287512-abc1-4756-abd2-236766a66782.png)

- Код соответствующего скрипта:

```py
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;

    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;

    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }

    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }

    public float forceMultiplier = 10;

    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if(distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}

```

- Добавление компонентов Decision Requester, Behavior Parameters для сферы:

![2022-10-26_00-55-08](https://user-images.githubusercontent.com/114309754/197883817-e630663a-e5ea-4aaa-a817-c57fb913b637.png)

- В корень проекта добавляем файл конфигурации нейронной сети, доступный в папке с файлами проекта. 

```py

behaviors:
  RollerBall:
    trainer_type: ppo
    hyperparameters:
      batch_size: 10
      buffer_size: 100
      learning_rate: 3.0e-4
      beta: 5.0e-4
      epsilon: 0.2
      lambd: 0.99
      num_epoch: 3
      learning_rate_schedule: linear
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 500000
    time_horizon: 64
    summary_freq: 10000
    
```

- Запускаем ml-агента. Запускаем сцену в Unity и проверяем работу ML-Agent.

![2022-10-27_22-32-17](https://user-images.githubusercontent.com/114309754/198359306-5786de41-366e-45f2-bbd9-d13373bff7cb.png)

- Создаем копии модели  «Плоскость-Сфера-Куб» 3, 9, 27, а также запускаем их симуляцию.

![3](https://user-images.githubusercontent.com/114309754/198360596-b675c6eb-88c4-469a-b7b4-60f0f4670ad0.png)
![9](https://user-images.githubusercontent.com/114309754/198360875-e1766c96-0662-40a2-ac1d-651ac4170740.png)
![27](https://user-images.githubusercontent.com/114309754/198360587-a4c70f2b-592a-4dac-b65e-c4520cad0e09.png)

 - Вывод: увеличение количества копий модели, выполняющих действие одновременно, помогает сделать обучение быстрее.

## Задание 2
### Подробно опишите каждую строку файла конфигурации нейронной сети, доступного в папке с файлами проекта по ссылке. Самостоятельно найдите информацию о компонентах Decision Requester, Behavior Parameters, добавленных сфере.

```py

behaviors:
  RollerBall:
    trainer_type: ppo
    hyperparameters:
      batch_size: 10
      buffer_size: 100
      learning_rate: 3.0e-4
      beta: 5.0e-4
      epsilon: 0.2
      lambd: 0.99
      num_epoch: 3
      learning_rate_schedule: linear
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    max_steps: 500000
    time_horizon: 64
    summary_freq: 10000
    
```
- trainer_type: ppo - тип обучения с поощрением.
- hyperparameters: Гиперпараметр модели.
- batch_size: Определяет количество информации, собираемой в ходе одного шага в градиентном спуске. Связан с параметром buffer_size, всегда должен быть его долей.
- buffer_size: Количество опыта, который необходимо собрать перед обновлением модели политики. Соответствует тому, сколько опыта должно быть собрано, прежде чем мы приступим к какому-либо изучению или обновлению модели. Это должно быть в несколько раз больше, чем batch_size. Обычно больший размер буфера соответствует более стабильным обновлениям обучения.
- learning_rate: Параметр задает величину каждого шага обновления градиентного спуска. Обычно это значение следует уменьшить, если обучение нестабильно, а вознаграждение не увеличивается последовательно.
- beta: Параметр задает степень регуляризации энтропии, и определяет, какой будет степень случайности в обновленной политике. Чем выше этот параметр, тем больше случайных действий будет выполнять модель. При оптимальном подборе параметра энтропия будет уменьшаться с увеличением среднего вознаграждения.
- epsilon: Влияет на то, насколько быстро политика может развиваться во время обучения. Соответствует допустимому порогу расхождения между старой и новой политиками при обновлении с градиентным спуском. Установка этого значения небольшим приведет к более стабильным обновлениям, но также замедлит процесс обучения.
- lambd: Обозначает то, насколько агент полагается на текущую оценку значения при расчете обновленной оценки значения. Чем выше значение, тем выше зависимость от фактических вознаграждений, полученных извне,что может привести к высокой дисперсии.
- num_epoch: Количество проходов, которые необходимо выполнить через буфер опыта при выполнении оптимизации градиентного спуска.Чем больше размер пакета, тем больше это допустимо сделать. Уменьшение этого параметра обеспечит более стабильные обновления за счет более медленного обучения.
- learning_rate_schedule : Определяет, как скорость обучения меняется с течением времени.
- network_settings: Параметры нейронной сети.
- normalize: К входным данным векторного наблюдения не будет применяться нормализация. Нормализация может быть полезна в случаях со сложными задачами непрерывного управления, но может быть вредна при более простых задачах дискретного управления.
- hidden_units: Соответствует количеству единиц в каждом полностью подключенном слое нейронной сети. Для простых задач, где правильным действием является простая комбинация входных данных наблюдения, это должно быть небольшим. Для задач, где действие представляет собой очень сложное взаимодействие между переменными наблюдения, это должно быть больше.
- num_layers: Определяет количество слоев нейронной сети. Для простых задач меньшее количество слоев, скорее всего, будет обучаться быстрее и эффективнее. Для более сложных задач управления может потребоваться больше уровней.
- reward_signals: Параметры вознаграждений.
- gamma: Параметр задает степень значимости будущих вознаграждений. Чем выше значение, тем больше агент должен действовать в настоящем, чтобы подготовиться к вознаграждению в отдаленном будущем.
- strength: Фактор, на который можно умножить вознаграждение, получаемое от окружающей среды. Типичные диапазоны будут варьироваться в зависимости от сигнала вознаграждения.
- max_steps: Максимальное количество проходов.
- time_horizon: Временной отрезок.
- summary_freq: Суммированная частота.
- DecisionRequester запрашивает принятие решения на основе сделанных наблюдений через определенные промежутки времени. Иными словами, DecisionRequester определяет, сколько шагов Академии должно быть выполнено, прежде чем будет запрошено решение.
- BehaviorParameters определяет, сколько наблюдений мы принимаем и какую форму будут принимать выводимые действия.
- BehaviorType имеет три варианта: эвристический (heuristic), по умолчанию (default) и вывод (inference). При установке значения по умолчанию, если нейронная сеть была сгенерирована, агент будет выполнять Inference, поскольку он использует нейронную сеть для принятия решений. Когда нейронная сеть не предусмотрена, она будет использовать эвристику. Эвристический метод можно рассматривать как традиционный подход к искусственному интеллекту, при котором программист вводит все возможные команды непосредственно в объект. Если установлено значение Heuristic only, агент будет запускать все, что находится в эвристическом методе.


## Задание 3
### Доработайте сцену и обучите ML-Agent таким образом, чтобы шар перемещался между двумя кубами разного цвета. Кубы должны, как и впервом задании, случайно изменять кооринаты на плоскости. 

- Код нового С#-скрипта:

```py

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public GameObject Target;
    public GameObject Target1;
    private bool touch_target;
    private bool touch_target1;

    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.transform.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
        Target1.transform.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
        Target.SetActive(true);
        Target1.SetActive(true);
        touch_target = false;
        touch_target1 = false;
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.transform.localPosition);
        sensor.AddObservation(Target1.transform.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(touch_target);
        sensor.AddObservation(touch_target1);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }

    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)

    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.transform.localPosition);
        float distanceToTarget1 = Vector3.Distance(this.transform.localPosition, Target1.transform.localPosition);

        if (!touch_target & distanceToTarget < 1.42f)
        {
            touch_target = true;
            Target.SetActive(false);
        }

        if (!touch_target1 & distanceToTarget1 < 1.42f)
        {
            touch_target1 = true;
            Target1.SetActive(false);

        }

        if (touch_target & touch_target1)
        {
            SetReward(1.0f);
            EndEpisode();
        }

        else if (this.transform.localPosition.y < 0)
        {
            SetReward(-0.5f);
            EndEpisode();
        }
    }
}

```
- Работа модели:

![1](https://user-images.githubusercontent.com/114309754/198362163-55f4208f-96a8-4464-8730-5bf46bfe7667.png)
![2](https://user-images.githubusercontent.com/114309754/198362155-d3429064-c9c5-4bcc-a452-0054398ed73c.png)

## Выводы

В ходе выполонения данной лабораторной работы я ознакомилась с основами использования ML агента для оптимизации траектории движения интеллектуального агента.

Игровой баланс - соблюдение равновесия между различными экономическими показателями в игре. Это включает в себя настройку сложности, условий выигрыша / проигрыша, игровых состояний, баланса экономики и так далее, чтобы работать в тандеме друг с другом. В таком случае системы машинного обучения можно использовать для создания интеллектуальных агентов с оптимальной сложностью прохождения для игрока или подбора коэффициентов в экономической системе со множеством параметров.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
