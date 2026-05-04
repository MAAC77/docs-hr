# Mapeo de Usuarios a Dispositivos

---

## Objetivo

Explicar cómo relacionar un usuario del dispositivo biométrico con un usuario del sistema.

---

## A quién aplica

Este manual aplica al personal con rol `Administrador`.

---

## Ruta de acceso

1. Ingresa al sistema.
2. En el menú lateral, abre `Sistema`.
3. Haz clic en `Mapeo de Usuarios`.

Ruta habitual: `/hr/devices/mappings`

---

## Qué verás en esta pantalla

En esta pantalla verás un listado de mapeos ya registrados.

La tabla normalmente muestra:

- `Usuario de Dispositivo`;
- `Usuario del Sistema`;
- `Dispositivo`;
- `Fecha de Mapeo`;
- `Acciones`.

También encontrarás:

- cuadro de búsqueda;
- botón `Nuevo Mapeo`.

---

## Para qué sirve este módulo

Este módulo se usa para decirle al sistema qué persona registrada en el biométrico corresponde a qué persona registrada en la plataforma.

Sin este mapeo, puede ocurrir que:

- existan marcaciones en el dispositivo;
- pero no puedan asociarse correctamente a una persona del sistema.

---

## Cómo crear un mapeo nuevo

1. Haz clic en `Nuevo Mapeo`.
2. En el primer selector, busca y selecciona el `Usuario de Dispositivo`.
3. Revisa la descripción que acompaña al registro del dispositivo.
4. En el segundo selector, busca y selecciona el `Usuario del Sistema`.
5. Revisa nuevamente que ambas personas correspondan realmente al mismo empleado.
6. Haz clic en `Crear Mapeo`.

---

## Qué revisar antes de guardar

Antes de crear un mapeo:

1. confirma el nombre de la persona en el dispositivo;
2. revisa el correo o nombre completo del usuario del sistema;
3. verifica que ambos registros correspondan a la misma persona;
4. evita crear relaciones apresuradas solo por coincidencia de nombre.

---

## Cómo editar un mapeo

1. Busca el mapeo en la tabla.
2. En `Acciones`, selecciona `Editar`.
3. Corrige los datos necesarios.
4. Revisa la nueva relación.
5. Guarda los cambios.

---

## Cómo eliminar o desactivar un mapeo

1. Busca el registro en la tabla.
2. Abre `Acciones`.
3. Selecciona la opción para eliminar o desactivar el mapeo.
4. Lee el mensaje de confirmación.
5. Confirma solo si estás seguro de que el mapeo ya no debe usarse.

---

## Cuándo conviene crear o corregir un mapeo

Usa este módulo cuando:

- se agregó un dispositivo nuevo;
- un empleado nuevo ya marca en el equipo;
- una persona aparece con marcaciones, pero no refleja asistencia correctamente;
- se detectó una asociación incorrecta.

---

## Errores o situaciones frecuentes

### No encuentras el usuario del dispositivo

Revisa:

1. si el dispositivo ya fue sincronizado;
2. si el usuario existe realmente en el equipo;
3. si estás filtrando correctamente.

### No encuentras el usuario del sistema

Revisa:

1. si la persona ya existe en `Usuarios`;
2. si el nombre o correo están bien escritos;
3. si la cuenta fue cargada en el sistema.

### El mapeo quedó mal

Si el mapeo es incorrecto:

1. corrígelo inmediatamente;
2. vuelve a revisar la asistencia o las marcaciones afectadas;
3. confirma con RRHH si hubo impacto en reportes.

---

## Resultado esperado

Al finalizar este proceso, el sistema debe poder relacionar correctamente las marcaciones biométricas con la persona correcta.

---

## Imágenes recomendadas

- pantalla principal del listado de mapeos;
- modal `Crear Nuevo Mapeo`;
- ejemplo del selector de usuario de dispositivo;
- ejemplo del selector de usuario del sistema.
