---
theme: default
highlighter: shiki
lineNumbers: false
title: Nuance on Kubernetes
info: |
  Сейчас контейнеризация и Kubernetes в частности — стандарт де-факто для запуска приложений «в бою». И запустить-то приложение в «кубе» несложно, но как всегда есть нюанс и не один. Обсудим, что нужно разработчику и админу учесть и сделать для того, чтобы приложение работало быстро и надёжно, не требуя к себе особого внимания. Например, посмотрим, как работают requests и limits на ресурсы, чем должны отличаться liveness и readiness пробы, и на что следует обращать внимание в мониторинге и так далее.

drawings:
  persist: false

fonts:
  provider: 'none'
  fallback: false
  local: 'Martian Grotesk,Martian Mono'
  sans: 'Martian Grotesk'
  serif: 'Martian Grotesk'
  mono: 'Martian Mono'

---

# <span class="wobbling">Нюансы</span><span class="block text-xl ultra-expanded">запуска приложений в</span><span class="ultra-condensed">Kubernetes</span>

<div class="absolute bottom-0 my-2">
Новиков Андрей, Злые марсиане<br />
<small><a href="https://www.trampoline.to/event/trampoline-8">Trampoline meetup №8</a></small><br />
<small><time datetime="2022-02-24">24 февраля 2022</time></small>
</div>

<div class="absolute bottom-0 right-0 h-40 scaled-image flex">
  <a href="https://evilmartians.com/" class="w-40 h-40 p-3"><img alt="Evil Martians" src="/images/01_Evil-Martians_Logo_v2.1_RGB.svg" /></a>
  <a href="https://www.trampoline.to/" class="w-40 h-40 p-4"><img alt="Trampoline" src="/images/trampoline-logo-512x512.png" /></a>
</div>

<style>
  a { border-bottom: none !important; }

  .wobbling {
    animation-duration: 1s;
    animation-name: wobbling;
    animation-iteration-count: infinite;
    animation-direction: alternate;
    animation-direction: alternate;
  }

  .ultra-condensed {
    font-stretch: 3%;
  }

  .ultra-expanded {
    font-stretch: 9%;
  }

  @keyframes wobbling {
    from {
      font-weight: 40;
      font-stretch: 9%;
    }

    to {
      font-weight: 200;
      font-stretch: 7%;
    }
  }
</style>

<!--
Начнём сегодня вечер с довольно хардкорной темы, пока вы свеженькие. Посмотрим, как можно хитро стрелять себе в ноги с помощью настроек Kubernetes, чтобы, разумеется, оставить свои ноги в целости и сохранности.
-->

---
layout: full
---

<a href="https://github.com/Envek/"><img src="/images/github-envek.png" class="object-contain text-center m-auto h-full w-full max-h-full max-w-full" /></a>

<!--
Всем привет! Меня зовут Андрей…
-->

---

<a href="https://evilmartians.com/?utm_source=trampoline&utm_medium=slides&utm_campaign=nuance-on-kubernetes"><img alt="Evil Martians" src="/images/01_Evil-Martians_Logo_v2.1_RGB.svg" class="object-contain text-center m-auto max-h-112"/></a>

<!--
…и я Марсианин.
-->

---

# Марсианский Open Source

<div class="grid grid-cols-4 grid-rows-2 gap-4">
  <a href="https://github.com/yabeda-rb/yabeda">
    <figure>
      <img alt="Yabeda" src="/images/martian-oss/yabeda.svg" class="scaled-image h-40 mx-auto" />
      <figcaption>Yabeda: Ruby application instrumentation framework</figcaption>
    </figure>
  </a>
  <a href="https://github.com/evilmartians/lefthook">
    <figure>
      <img alt="LeftHook" src="/images/martian-oss/lefthook.svg" class="scaled-image h-40 mx-auto" />
      <figcaption>Lefthook: git hooks manager</figcaption>
    </figure>
  </a>
  <a href="https://anycable.io/">
    <figure>
      <img alt="AnyCable" src="/images/martian-oss/anycable.svg" class="scaled-image h-40 mx-auto" />
      <figcaption>AnyCable: Polyglot replacement for ActionCable server</figcaption>
    </figure>
  </a>
  <a href="https://postcss.org/">
    <figure>
      <img alt="PostCSS" src="/images/martian-oss/postcss.svg" class="scaled-image h-40 mx-auto" />
      <figcaption>PostCSS: A tool for transforming CSS with JavaScript</figcaption>
    </figure>
  </a>
  <a href="https://imgproxy.net/">
    <figure>
      <img alt="Imgproxy" src="/images/martian-oss/imgproxy.svg" class="scaled-image h-40 mx-auto" />
      <figcaption>Imgproxy: Fast and secure standalone server for resizing and converting remote images</figcaption>
    </figure>
  </a>
  <a href="https://logux.io/">
    <figure>
      <img alt="Logux" src="/images/martian-oss/logux.svg" class="scaled-image h-40 mx-auto" />
      <figcaption>Logux: Client-server communication framework based on Optimistic UI, CRDT, and log</figcaption>
    </figure>
  </a>
  <a href="https://github.com/DarthSim/overmind">
    <figure>
      <img alt="Overmind" src="/images/martian-oss/overmind.svg" class="scaled-image h-40 mx-auto" />
      <figcaption>Overmind: Process manager for Procfile-based applications and tmux </figcaption>
    </figure>
  </a>
  <a href="https://evilmartians.com/oss">
    <figure>
      <div class="h-40 text-2xl flex items-center justify-center">
        <qr-code-vue value="https://evilmartians.com/oss" class="scaled-image w-full h-full mx-auto p-4" render-as="svg" />
      </div>
      <figcaption style="font-size: 0.5rem;">И многие другие на evilmartians.com/oss</figcaption>
    </figure>
  </a>
</div>

<style>
  a { border-bottom: none !important; }
  figcaption {
    font-size: 0.4rem;
    line-height: 1rem;
  }
</style>

<!--
Марсиане зарабатывают себе на хлеб в первую очередь заказной разработкой и консалтингом для разных клиентов, в основном штатовских стартапов, но нас вы скорее всего знаете по нашим опенсорс-проектам, которыми мы бесконечно гордимся и которые были извлечены из недр коммерческой разработки и мы усиленно используем их везде, где только можно. Используйте и вы.

Кстати, вот этот проект, сверху слева, Ябеда — моё детище. И он имеет некоторое отношение к Kubernetes. Скоро посмотрим, какое.
-->

---
layout: full
---

<a href="https://evilmartians.com/?utm_source=trampoline&utm_medium=slides&utm_campaign=nuance-on-kubernetes"><img alt="Evil Martians" src="/images/Evil-Martians_Logo_Kubernetes.svg" class="text-center m-auto"/></a>

<!--
Ну вот, мы все такие злые и можем в Кубернетес. Любим, умеем, практикуем, как говорится.
-->

---
layout: intro
---

# Kubernetes 101

Шо, какой ещё Kubernetes?

---

## Зачем разработчику знать Kubernetes?

TL;DR: Чтобы быстрее деплоить новые фичи самим и не ждать админов/девопсов.

- [Kubernets'а бояться — в деплой не ходить](https://youtu.be/bOpBWZt9HPM) от марсианского SRE.

  <div class="flex">
  <Youtube id="bOpBWZt9HPM" class="max-h-36" />
  <qr-code url="https://youtu.be/bOpBWZt9HPM" class="ml-16 w-36 h-36 px-0 py-0" />
  </div>

- [Вечерняя школа Kubernetes для разработчиков](https://www.youtube.com/playlist?list=PL8D2P0ruohOBSA_CDqJLflJ8FLJNe26K-) от Slurm.io

  <div class="flex">
  <Youtube id="Mw_rEH2pElw" class="max-h-36" />
  <qr-code url="https://www.youtube.com/playlist?list=PL8D2P0ruohOBSA_CDqJLflJ8FLJNe26K-" class="ml-16 w-36 px-0 py-0" />
  </div>

---
layout: footnote
footnoteClass: text-sm
---

## Что такое Kubernetes…

- **Кластерная операционная система** для деплоя приложений

- Абстрагирует приложение от нижележащего железа/облаков\*

- Декларативная конфигурация и встроенный control loop

- Использует (Docker-)контейнеры для запуска приложений…

- …и свои абстракции для их оркестрации и запуска

::footnote::

\* https://buttondown.email/nelhage/archive/two-reasons-kubernetes-is-so-complex/

---
layout: footnote
footnoteClass: relative
---

## …и из чего он состоит

<a href="https://www.slideshare.net/KirillKouznetsov/how-to-learn-k8s-for-developers/27" target="_blank">
<img src="/images/how-to-learn-k8s-for-developers-27-1024.webp" class="scaled-image max-h-104 text-center mx-auto" />
</a>

::footnote::

Источник: [Kubernets'а бояться — в деплой не ходить (слайды)](https://www.slideshare.net/KirillKouznetsov/how-to-learn-k8s-for-developers/)

<qr-code url="https://www.slideshare.net/KirillKouznetsov/how-to-learn-k8s-for-developers/27" class="absolute bottom-0 right-0 w-36 max-h-36" />

---
layout: two-cols
class: relative
---

## Pod

 - минимальная и главная единица в k8s — «атом» ⚛️

 - логически неделимая группа контейнеров (но обычно 1 основной)

 - запускаются вместе на одной машине

 - делят между собой localhost и сеть

 - есть внутрикластерный IP-адрес

 - с точки зрения приложения — как будто отдельный сервер

<div class="absolute bottom-0 text-sm">
Документация: <a href="https://kubernetes.io/docs/concepts/workloads/pods/" target="_blank">kubernetes.io/docs/concepts/workloads/pods</a>
</div>

::right::

<img src="/images/kubernetes_nodes_pods.svg" class="scaled-image">


<div class="absolute bottom-0 text-sm">
Картинка: <a href="https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/" target="_blank">kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro</a>
</div>

---
layout: two-cols
class: relative
---

## Service

 - Абстракция для логической группировки pod'ов

 - Service discovery — есть внутрикластерное DNS-имя

 - Балансируют трафик между своими подами (round robin)

 - Реализованы как набор правил iptables

 - Позволяет нам масштабировать приложения горизонтально

<div class="absolute bottom-0 text-sm">
Документация: <a href="https://kubernetes.io/docs/concepts/services-networking/service/" target="_blank">kubernetes.io/docs/concepts/services-networking/service</a>
</div>

::right::

<img src="/images/kubernetes_services.svg" class="scaled-image mt-16">

<div class="absolute bottom-0 text-sm">
Картинка: <a href="https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/" target="_blank">kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro</a>
</div>

<!--

Под включается в сервис, когда:

 - его labels совпали с matchlabels у сервиса

 - у него успешно проходит readiness check (если есть)

-->

---
layout: intro
---

# Healthcheck'и

зачем их столько и в чём разница?

---
layout: footnote
footnoteClass: text-sm
---
## Kubernetes health probes

У каждого отдельного **контейнера** внутри каждого пода:

 - liveness

   Контейнер убивается и перезапускается, если он не отвечает на «ты жив?».

 - readiness

   Под исключается из балансировки трафика через сервис, если не отвечает на «ну что, готов?» и включается обратно, когда отвечает утвердительно.

 - startup (k8s 1.20+)

   Позволяет отсрочить начало liveness и readiness проверок для долгозапускающихся приложений

<div class="text-base border border-2 border-green-600 rounded-lg px-4 my-2">

**Важно**: и liveness и readiness выполняются параллельно всё время жизни пода.

</div>

::footnote::

https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/


---
layout: two-cols
class: relative
---

## Просто (и неправильно)

Одинаковые пробы для веб-приложения:

```yaml {all|3-8|9-14|5-6,11-12}
containers:
  - name: app
    livenessProbe:
      httpGet:
        path: /health
        port: 80
      timeoutSeconds: 3
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /health
        port: 80
      timeoutSeconds: 3
      periodSeconds: 10
```

::right::

<img src="/images/ctrl-c-ctrl-v-dump-2016.png" class="scaled-image p-12">

<div class="absolute bottom-0 text-xs">
Картинка: <a href="https://www.behance.net/gallery/35173039/Stickers-for-another-one-IT-conference-(DUMP2016)" target="_blank">behance.net/gallery/35173039/Stickers-for-another-one-IT-conference-(DUMP2016)</a>
</div>


---
layout: image
image: /images/what-is-the-difference.jpg
class: relative
---

<div class="absolute bottom-4 text-2xl text-center w-full">
А какая разница?
</div>

---
layout: footnote
footnoteClass: text-sm
---

## Очереди запросов

Запросы ждут свободного воркера/бэкенда в Nginx или апп-сервере.

<img src="/images/puma-request-queue.png" class="scaled-image text-center mx-auto max-h-96">

::footnote::

Картинка: https://railsautoscale.com/request-queue-time/

---
class: text-xl
---

## Приходит нагрузка 🏋️

 1. Медленные запросы попадают на один под и «забивают» его

 2. «Забитый» контейнер в поде перестаёт отвечать на liveness 🥀

 3. Kubernetes убивает контейнер 💀

 4. И тут же запускает его заново, но это занимает какое-то время… ⌚

 4. На время перезапуска на другие поды приходит больше запросов.

 5. GOTO 1 🤡

<div class="text-xl border border-2 border-red-600 rounded-lg p-4 my-12">
Неправильно настроенная liveness-проба под нагрузкой убьёт приложение, под за подом!
</div>

---
layout: image
image: /images/death-knocking-on-doors.png
class: relative
---

<div class="absolute bottom-0 text-sm">
Картинка: https://knowyourmeme.com/photos/1901279-death-knocking-on-doors
</div>

---
layout: two-cols
class: relative
---

## Что же делать?

Пустить liveness пробу в обход!

```yaml {6}
containers:
  - name: app
    livenessProbe:
      httpGet:
        path: /health
        port: 8080 # ← другой порт
      timeoutSeconds: 3
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /health
        port: 80
      timeoutSeconds: 3
      periodSeconds: 10
```

::right::

<img src="/images/do-it-the-right-way.png" class="scaled-image p-12">

<div class="absolute bottom-0 text-xs">
Картинка: <a href="https://www.behance.net/gallery/35173039/Stickers-for-another-one-IT-conference-(DUMP2016)" target="_blank">behance.net/gallery/35173039/Stickers-for-another-one-IT-conference-(DUMP2016)</a>
</div>

---

## Healthcheck'и: итого

 1. **Liveness ходит через «чёрный ход»**

    Поднимите listener на отдельном порту, куда будет ходить только проба.

    <div class="text-xl border border-2 border-red-600 rounded-lg p-4 my-4">
    Kubernetes не должен прибивать ваши поды под нагрузкой!
    </div>

 2. **Readiness ходит вместе с клиентскими запросами**

    Позвольте «забитому» поду выйти из балансировки и «остыть».

    Таймаут нужно подобрать эмпирически, он не должен быть слишком мал.

<div class="text-xl border border-2 border-green-600 rounded-lg p-4 my-12">
<strong>Мораль</strong>: делайте стресс-тесты!
</div>


---
layout: footnote
footnoteClass: text-sm
---

## Следите за очередью запросов!

**Время ожидания запросов в очереди — это главная метрика**, которая показывает, что приложение «на пределе». **Выведите её себе в мониторинг.**

- Если она ощутимо больше 0 — надо скейлиться вверх (в Kubernetes есть Horizontal Pod Autoscaler)

- Если она всегда строго 0 — можно подумать о скейлинге вниз.

<div class="text-sm border border-2 border-green-600 rounded-lg p-4 my-8">
Следите за USE и RED метриками, когда дело касается производительности!

 - Utilization: количество свободных воркера
 - Saturation: время ожидания свободного воркера (95p)
 - Errors: процент ошибок при обработке запросов
 - Duration: время обработки запроса (95p)

</div>


::footnote::

USE method: https://www.brendangregg.com/usemethod.html

RED method: https://grafana.com/blog/2018/08/02/the-red-method-how-to-instrument-your-services/

<!--
Если запросы у вас в очереди никогда не ждут — у вас слишком много серверов приложений и вы зря ускоряете глобальное потепление (и  переплачиваете за инфраструктуру).
-->

---
layout: footnote
footnoteClass: text-sm
---

## А как масштабироваться-то?

<p class="my-8 text-xl">Kubernetes Horizontal Pod Autoscaler!</p>

<div class="flex">
<Youtube id="B1j5SdzlRPQ" />
<qr-code url="https://youtu.be/B1j5SdzlRPQ" class="ml-24 w-48 h-48 px-0 py-0" caption="Видео" />
<qr-code url="https://speakerdeck.com/dragonsmith/razbiraiem-avtoskieilingh-v-kubernetes" class="ml-24 w-48 h-48 px-0 py-0" caption="Слайды" />
</div>

<div class="border border-2 border-green-600 rounded-lg p-4 my-12">
Используйте USE-метрики для масштабирования!
</div>


::footnote::

Слайды: https://speakerdeck.com/dragonsmith/razbiraiem-avtoskieilingh-v-kubernetes

---
layout: intro
---

# Requests и limits

Как (не)правильно просить ресурсы и у кого.

---
layout: footnote
footnoteClass: text-xs relative
---

## Requests и limits 101

Для каждого контейнера в Pod'е:

```yaml
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

requests и limits для Pod — сумма значений его контейнеров.

requests — инструкции для планировщика k8s

limits — инструкции для ядра ОС на нодах кластера

cpu — измеряются в millicpu (тысячных долях процессорного ядра)

memory — измеряются в байтах и кратных (мебибайты, гебибайты)

::footnote::

https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

<qr-code url="https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/" class="w-36 px-0 absolute bottom-0 right-0" />

---
layout: two-cols
class: relative
---

## Requests 101

Инструкции для планировщика k8s для распределения Pod'ов на ноды:

```yaml {2-4}
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

 - Не используются в рантайме\*
 - Не учитывают фактическое потребление\*, только requests других подов

<div class="absolute bottom-0 text-xs">

\* Используются при настройке OOM, но про это дальше

</div>

::right::

<img src="/images/requests-and-scheduling.png" class="scaled-image p-12">

<div class="absolute bottom-0 text-xs pl-12">

Картинка: [Илья Черепанов](https://github.com/deponian), внутренняя школа k8s

</div>


---
layout: two-cols
class: relative
---

## Limits 101

Реализуются ядром ОС на нодах:

```yaml {5-7}
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 400m
    memory: 512Mi
```

 - cpu настраивает троттлинг CPU

 - memory настраивает Out of Memory Killer


---
layout: footnote
footnoteClass: text-xs relative
---

## Так, что там за milliCPU ещё?

<div class="grid grid-cols-5 grid-rows-2 gap-4">

<div class="text-sm col-span-3">

В случае limits настраивает CPU throttling — долю процессорного времени, которое можно использовать в течение 100мс.

</div>

<img src="/images/unconstrainedCPU_limits.png" class="scaled-image col-span-2 max-h-36"/>

<div class="col-span-3 text-sm">

```yaml {6}
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 400m
    memory: 512Mi
```

</div>

<img src="/images/constrainedCPU_limits.png" class="scaled-image col-span-2 max-h-36"/>
</div>


<div class="border border-2 border-green-600 rounded-lg p-4 my-4 mr-36 text-sm">
Не указывайте дробные CPU limits для контейнеров, где важна скорость ответа!
</div>


::footnote::

Подробнее: https://engineering.indeedblog.com/blog/2019/12/unthrottled-fixing-cpu-limits-in-the-cloud/

<qr-code url="https://engineering.indeedblog.com/blog/2019/12/unthrottled-fixing-cpu-limits-in-the-cloud/" class="absolute bottom-0 right-0 px-0 max-h-32" />

---
layout: footnote
footnoteClass: text-xs relative
---

## Утилизируем ядра

<div class="grid grid-cols-2 gap-4">

<div>

<div class="text-sm border border-2 border-red-600 rounded-lg px-4 mb-4">

Процесс на языках с GIL (Ruby, Python) не может утилизировать процессорное ядро целиком! **Максимум на 30-50%.**

</div>

Используйте **и** процессы **и** потоки:

 - Запускайте 2-4 рабочих процесса на контейнер <small>(работает CoW 🐮)</small>

 - По 4-6 потоков в каждом

<div class="text-sm border border-2 border-green-600 rounded-lg px-4 my-4">

➕ Улучшается утилизация воркеров за счёт раздачи запросов простаивающим воркерам.

</div>

</div>

<img src="/images/puma-request-queue.png" class="scaled-image text-center mx-auto max-h-96">

</div>

::footnote::

Подробнее: https://www.speedshop.co/2017/10/12/appserver.html

<qr-code url="https://www.speedshop.co/2017/10/12/appserver.html" class="px-0 absolute bottom-0 right-0 max-h-32" />


<!--

Итак, мы выяснили, что надо выдавать контейнерам процессорные ядра целиком, чтобы избежать замедления из-за троттлинга. Теперь у нас новая проблема — как теперь эти ядра загрузить по полной, чтобы не переплачивать за простаивающие ресурсы?

Нюанс тут в том, что многие популярные языки высокого уровня, в том числе Ruby, который мы нежно любим и вовсю используем, на самом деле не являются полностью параллельными и, используя только потоки операционной системы, нельзя полностью нагрузить одно процессорное ядро. Хотя, в случае Ruby, один поток может выполнять Ruby-код, а остальные в это время могут ждать ответа от базы или от сети, обрабатывать XML и JSON нативными расширениями и, казалось бы, должны при высоком количестве тредов забивать процессор на 100%, но нет. На практике больше 50% увидеть сложно.

Решение для этого на самом деле старо как мир, но очень контр-интуитивно в эпоху контейнеризации — использовать старую гибридную многопроцессную и многотредовую схему, где мастер-процесс принимает запросы от клиентов и раздаёт их рабочим процессам, которые форкает от себя. Лимит и по памяти и по процессору действует на всю эту группу процессов разом и так мы можем получить и отсутствие троттлинга и полную утилизацию процессора. При этом ещё и память экономится за счёт техники copy-on-write ядра Linux.

Бонусом ещё и улучшается утилизация воркер-процессов, поскольку мастер-процесс знает, кто из воркеров сейчас занят, а кто простаивает и таким образом может передавать новый запрос именно свободному процессу. В случае, когда у нас в каждом контейнере по одному процессу и балансировка производится только силами кубового сервиса, который на самом деле простой, если не сказать тупой, набор правил к iptables, то сервис просто раздаёт коннекты round robin'ом и часто получается ситуация, когда часть процессов простаивают без дела, а часть заняты полностью и ещё копят запросы в очереди.

Таким образом, укрупнение контейнеров с веб-приложением может повысить пропускную способность и уменьшить задержку не только за счёт увеличения утилизации процессора, но и за счёт более равномерной утилизации самих серверов приложений, а это уменьшает задержку. Смотрите, сразу два нюанса на один слайд получилось!

Внутренний холивар про процессы vs контейнеры (на будущее): https://evilmartians.slack.com/archives/CBWBNDTHT/p1556540922007900

-->

---
layout: footnote
footnoteClass: text-xs relative
---

## Requests × Limits = QoS

В различных сочетаниях реквестов и лимитов поведение контейнера может меняться драматично:

<div class="grid grid-cols-5 gap-4">

<div class="text-sm col-span-3">

 1. `Guaranteed` — requests = limits
    - гарантированно выдаётся CPU
    - убиваются по OOM последними
 2. `Burstable` — requests ≠ limits
    - могут использовать CPU больше запрошенного
    - убиваются после BestEffort, в порядке «жадности»
 3. `BestEffort` — нет ни requests ни limits
    - CPU выдаётся в последнюю очередь
    - Первыми убиваются OOM-киллером

<div class="border border-2 border-green-600 rounded-lg p-4 my-4">
Всегда указывайте и requests и limits!
</div>


</div>

<img src="/images/qos-requests-limits.png" class="scaled-image col-span-2">

</div>

::footnote::

Подробнее: [Управление ресурсами в Kubernetes — habr.com/ru/company/flant/blog/459326/](https://habr.com/ru/company/flant/blog/459326/)

<qr-code url="https://habr.com/ru/company/flant/blog/459326/" class="px-0 absolute right-0 bottom-0 max-h-32" />

---
layout: intro
---

# Ну всё

Ну почти.

---
layout: footnote
---

## Нам нужны твои мозги!

<a href="https://evl.ms/jobs?utm_source=trampoline&utm_medium=slides&utm_campaign=nuance-on-kubernetes">
<img class="mx-auto text-center max-h-96" src="/images/evl-ms-jobs-qrcode-trampoline-slides.png" />
</a>

::footnote::

Пункт призыва на галактическую службу по контракту: [evl.ms/jobs](https://evl.ms/jobs?utm_source=trampoline&utm_medium=slides&utm_campaign=nuance-on-kubernetes)

<!-- Хотите узнать это и многое другое в бою, да ещё и с приличным жалованьем в придачу! Придётся присягнуть на верность Императору и отправиться покорять дальние миры в Галактике, но поверьте, оно того стоит!  -->

---

# Внимание! Внимание!
## Спасибо за внимание!

<div class="grid grid-cols-4 grid-rows-3 mt-16 gap-4">

<div class="justify-self-end">
<img alt="Andrey Novikov" src="https://secure.gravatar.com/avatar/d0e95abdd0aed671ebd0920c16d393d4?s=512" class="w-32 h-32 scaled-image" />
</div>

<ul class="list-none col-span-2">
<li><a href="https://github.com/Envek"><logos-github-icon /> @Envek</a></li>
<li><a href="https://t.me/envek"><logos-telegram /> @Envek</a></li>
<li><a href="https://twitter.com/Envek"><logos-twitter /> @Envek</a></li>
<li><a href="https://www.instagram.com/Envek"><logos-instagram-icon /> @Envek</a></li>
</ul>

<div>
<qr-code url="https://github.com/Envek" caption="github.com/Envek" class="w-36" />
</div>

<div class="justify-self-end row-span-2">
<a href="https://evilmartians.com/"><img alt="Evil Martians" src="/images/01_Evil-Martians_Logo_v2.1_RGB.svg" class="w-32 h-32 scaled-image" /></a>
</div>

<div class="row-span-2 col-span-2">

- <logos-telegram />  [@evilmartians](https://t.me/evilmartians?utm_source=trampoline&utm_medium=slides&utm_campaign=nuance-on-kubernetes)
- <logos-twitter /> [@evilmartians_ru](https://twitter.com/evilmartians_ru/?utm_source=trampoline&utm_medium=slides&utm_campaign=nuance-on-kubernetes)
- <logos-instagram-icon /> [@evil.martians](https://www.instagram.com/evil.martians/?utm_source=trampoline&utm_medium=slides&utm_campaign=nuance-on-kubernetes)

Вакансии: [evilmartians.com/jobs](https://evilmartians.com/jobs/?utm_source=trampoline&utm_medium=slides&utm_campaign=nuance-on-kubernetes)

Блог: [evilmartians.com/chronicles](https://evilmartians.com/chronicles/?utm_source=trampoline&utm_medium=slides&utm_campaign=nuance-on-kubernetes)
</div>

<div>
<qr-code url="https://taplink.cc/evil.martians" caption="Evil Martians: links" class="w-36" />
</div>
</div>

<style>
  ul a { border-bottom: none !important; }
  ul { list-style-type: none !important; }
  ul li { margin-left: 0; padding-left: 0; }
</style>
