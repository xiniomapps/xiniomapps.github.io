---
layout: post
title: Can't find gem cocoapods (>= 0.a) with executable pod (Gem::GemNotFoundException)
tags: [errors, ios]
---

Hoy mientras configuraba Firebase para iOS me encontré con este problema:

```text
Can't find gem cocoapods (>= 0.a) with executable pod (Gem::GemNotFoundException)
```

interesante ya que no hacía poco que había hecho una actualización de mis pods. La solución:

```text
gem install cocoapods
```

Mi sospecha es que tiene algo que ver con las Gemfiles que agregué para la configuración de Fastlane. Seguiré investigando.
