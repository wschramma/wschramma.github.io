---
layout: post
author: wschramm
title:  "Funciones útiles para fechas en MsSql"
date:   2020-05-21 15:20:23
categories: [sql]
tags: [sql]
---

1.- Para obtener el día de hoy

```sql
SELECT GETDATE()
```

2.- Para obtener la fecha de ayer

```sql
SELECT DATEADD(d,-1,GETDATE())
```

3.- Inicio del día actual
```sql
SELECT DATEADD(dd,DATEDIFF(dd,0,GETDATE()),0)
```

4.- Fin del día actual
```sql
SELECT DATEADD(ms,-3,DATEADD(dd,DATEDIFF(dd,0,GETDATE()),1))
```

5.- Inicio de ayer
```sql
SELECT DATEADD(dd,DATEDIFF(dd,0,GETDATE()),-1)
```

6.- Fin de ayer
```sql
SELECT DATEADD(ms,-3,DATEADD(dd,DATEDIFF(dd,0,GETDATE()),0))
```

7.- Primer día de la semana actual
```sql
SELECT DATEADD(wk,DATEDIFF(wk,0,GETDATE()),0)
```

8.- Último día de la semana actual
```sql
SELECT DATEADD(wk,DATEDIFF(wk,0,GETDATE()),6)
```


9.- Primer día de la semana pasada
```sql
SELECT DATEADD(wk,DATEDIFF(wk,7,GETDATE()),0)
```

10.- último día de la semana pasada
```sql
SELECT DATEADD(wk,DATEDIFF(wk,7,GETDATE()),6)
```

11.- Primer día del mes actual
```sql
SELECT DATEADD(mm,DATEDIFF(mm,0,GETDATE()),0)
```

12 .- Último día del mes actual
```sql
SELECT DATEADD(ms,-3,DATEADD(mm,0,DATEADD(mm,DATEDIFF(mm,0,GETDATE())+1,0)))
```

13.- Primer día del mes pasado
```sql
SELECT DATEADD(mm,-1,DATEADD(mm,DATEDIFF(mm,0,GETDATE()),0))
```

14.- Último día del mes pasado
```sql
SELECT DATEADD(ms,-3,DATEADD(mm,0,DATEADD(mm,DATEDIFF(mm,0,GETDATE()),0)))
```

15.- Primer día del año actual
```sql
SELECT DATEADD(yy,DATEDIFF(yy,0,GETDATE()),0)
```

16.- Ùltimo dìa del año actual
```sql
SELECT DATEADD(ms,-3,DATEADD(yy,0,DATEADD(yy,DATEDIFF(yy,0,GETDATE())+1,0)))
```

17.- Primer día del año pasado
```sql
SELECT DATEADD(yy,-1,DATEADD(yy,DATEDIFF(yy,0,GETDATE()),0)) 
```

18.- Último día del año pasado
```sql
SELECT DATEADD(ms,-3,DATEADD(yy,0,DATEADD(yy,DATEDIFF(yy,0,GETDATE()),0)))
```

19.- Primer día del proximo mes
```sql
SELECT DATEADD(mm,1,DATEADD(mm,DATEDIFF(mm,0,GETDATE()),0))
```

20.- Último día del próximo mes
```sql
SELECT DATEADD(ms,-3,DATEADD(mm,DATEADD(mm,(DATEDIFF(mm,0,GETDATE()),0)))
```
