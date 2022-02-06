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

# <span class="wobbling">Nuance</span> on Kubernetes

<div class="absolute bottom-0 my-2">
Новиков Андрей<br />
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

  @keyframes wobbling {
    from {
      font-weight: 40;
    }

    to {
      font-weight: 200;
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

TODO: Actual content[^1]

[^1]: huh?


<style>
.footnotes-sep {
  @apply mt-40 opacity-10;
}
.footnotes {
  @apply text-sm opacity-75;
}
.footnote-backref {
  display: none;
}
</style>

---

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
