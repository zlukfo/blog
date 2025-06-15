---
title: 
description: 
permalink: 
tags:
  - python
  - Битрикс24
draft: false
date:
---
Для хранения настроек и конфигураций в таблицах коробочной версии Битрикс24 применяются сериализованные php-массивы. Один из способов привести такой массив в человекочитаемый вид - использовать следующий код на Python

```python
from phpserialize import *
import json
data = str.encode(serialized_array)
data = loads(data, object_hook=phpobject,decode_strings=True)
print (json.dumps(data, ensure_ascii=False, indent=3))
```

