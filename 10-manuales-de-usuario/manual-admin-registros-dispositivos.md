# Registros de Dispositivos

---

## Objetivo

Explicar cómo revisar los registros recibidos desde los dispositivos para validar marcaciones y sincronización.

---

## A quién aplica

Este manual aplica al personal con rol `Administrador` y, cuando corresponda, al personal de `RRHH` con acceso autorizado.

---

## Ruta de acceso

1. Ingresa al sistema.
2. En el menú lateral, abre `Sistema`.
3. Haz clic en `Registros de Dispositivos`.

Ruta habitual: `/hr/devices/raw-records`

---

## Qué verás en esta pantalla

En esta pantalla verás el detalle de registros recibidos desde los dispositivos.

Normalmente encontrarás:

- filtros de fecha;
- búsqueda;
- datos del dispositivo;
- fecha y hora del registro;
- estado u otra información del evento.

---

## Para qué usar esta pantalla

Usa esta pantalla cuando necesites confirmar:

- si una marcación llegó desde el equipo;
- si un dispositivo está enviando registros;
- si una incidencia de asistencia puede venir desde el origen biométrico.

---

## Cómo buscar registros

1. Abre la pantalla.
2. Revisa si existen filtros de fecha.
3. Selecciona el rango de fechas que deseas revisar.
4. Si el sistema muestra búsqueda, escribe el dato disponible, por ejemplo nombre o identificador.
5. Espera a que la tabla se actualice.
6. Revisa los resultados obtenidos.

---

## Cómo validar una incidencia con esta pantalla

Si te reportan una marcación faltante:

1. identifica la persona afectada;
2. identifica la fecha aproximada;
3. busca registros del período;
4. confirma si existe un evento en el dispositivo;
5. anota la hora y el equipo que generó el registro.

Si el registro existe aquí, pero no aparece correctamente en asistencia, el siguiente paso es revisar el procesamiento del sistema y el mapeo del usuario.

---

## Diferencia entre registro y asistencia

Recuerda:

- el `registro de dispositivo` es el dato original recibido del equipo;
- la `asistencia` es el resultado procesado por el sistema.

Por eso, puede existir un registro en esta pantalla y, aun así, requerirse revisión adicional en asistencia.

---

## Qué revisar antes de reportar un problema

1. confirma el rango de fechas;
2. revisa si el dispositivo correcto fue sincronizado;
3. valida si el usuario tiene mapeo correcto;
4. confirma si el dato esperado es realmente una marcación y no un resultado procesado.

---

## Errores o situaciones frecuentes

### No aparecen registros

Revisa:

1. si el período seleccionado es correcto;
2. si el dispositivo fue sincronizado;
3. si el equipo está activo;
4. si el problema corresponde realmente a otro módulo.

### Hay registros, pero no aparecen en asistencia

En ese caso revisa:

1. el mapeo del usuario;
2. la asistencia del período;
3. si existe alguna incidencia de horario o procesamiento.

---

## Resultado esperado

Al finalizar la revisión, debes poder confirmar si el dato existe en el origen biométrico y si hace falta seguir investigando en otros módulos.

---

## Imágenes recomendadas

- pantalla principal de registros de dispositivos;
- zona de filtros de fecha;
- ejemplo de una fila de registro con hora y dispositivo visibles.
