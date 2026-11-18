# bold-mode

**Ideas arriesgadas, sin poner en riesgo tu código.**

[English](README.md)

Un plugin para Claude Code que permite a Claude intentar una idea audaz —por ejemplo, borrar una capa entera o preguntar “¿y si esto no existiera?”— sin poner en peligro el código que funciona. Cada experimento se ejecuta en una copia aislada, con un presupuesto de pasos rígido, y termina con una tabla comparativa honesta frente al estado existente.

## Qué incluye

- **Skill `bold-mode`:** aislamiento, medición de la línea base, tres ideas (segura, audaz y arriesgada), construcción, medición, informe y paso a revisión.
- **11 lentes:** atajos de pensamiento para encontrar ideas no evidentes.
- **hook guardián:** activo solo durante una ejecución bold. Durante esa ejecución:
  - cuenta pasos y detiene la ejecución cuando se agota el presupuesto (valor por defecto 25),
  - bloquea la edición de los archivos originales,
  - bloquea force-push, push a main, publicación, despliegue y eliminación de datos.
- **Comando `/bold`:** para activar en una sola línea.

## Instalación

```
/plugin marketplace add cladueusueer/bold-mode
/plugin install bold-mode@bold-mode
```

Después de la instalación hay que reiniciar Claude Code. El hook requiere `python3`.

## Uso

```
/bold capa de caché en src/cache
```

## Cuándo no usarlo

Para correcciones pequeñas, bugs, faltas de ortografía, refactorizaciones rutinarias o código estable que ya funciona.

## Configuraciones

- `BOLD_MODE_BUDGET`: número máximo de pasos (valor por defecto 25).
- `BOLD_MODE_GUARD=off`: desactiva todos los bloqueos y el límite de pasos.

## Limitaciones honestas

- El hook ve llamadas a herramientas, no intenciones. Un comando Bash que escriba en el original mediante redirección (`>`), o un script que lo haga, no será detectado. Las instrucciones del skill y la copia aislada son la protección principal; el hook es el cinturón de seguridad.
- Los patrones de comando actúan como una lista de bloqueo, no como una sandbox. Si necesitas garantías estrictas, ejecuta Claude Code en un contenedor.
- “Medido” solo es tan bueno como tus tests. Sin tests, el skill crea algunas pruebas rápidas y la comparación es menos concluyente.

## Contribuir

La forma más fácil de contribuir es añadir una nueva lente: un solo archivo markdown corto. Más detalles en [CONTRIBUTING.md](CONTRIBUTING.md).
