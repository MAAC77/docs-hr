# Gestión de Usuarios

---

## Objetivo

Explicar cómo consultar usuarios, editar información del sistema y realizar importaciones masivas por archivo CSV.

---

## A quién aplica

Este manual aplica al personal con rol `Administrador`.

---

## Ruta de acceso

1. Ingresa al sistema.
2. En el menú lateral, abre `Sistema`.
3. Haz clic en `Usuarios`.

Ruta habitual: `/hr/admin/users`

---

## Qué verás en esta pantalla

En esta pantalla verás el listado de usuarios del sistema.

Normalmente encontrarás:

- tabla de usuarios;
- búsqueda por texto;
- acciones por registro;
- botón `Importar CSV`.

---

## Cómo buscar un usuario

1. Abre la pantalla `Usuarios`.
2. En el cuadro de búsqueda, escribe nombre, apellido, correo, documento u otro dato disponible.
3. Espera a que la tabla se actualice.
4. Revisa si el usuario aparece en el listado.

Usa la búsqueda antes de editar o importar, para evitar duplicaciones o revisiones innecesarias.

---

## Cómo editar información del sistema

En este módulo no estás corrigiendo todos los datos personales del empleado. Aquí normalmente editarás datos del sistema, como la información administrativa asociada al usuario.

1. Busca el usuario en la tabla.
2. En la columna `Acciones`, selecciona la opción para editar.
3. Revisa la información cargada en el formulario.
4. Corrige los campos necesarios.
5. Haz clic en `Guardar cambios`.

---

## Qué revisar antes de guardar

1. confirma que estás editando a la persona correcta;
2. revisa que el dato nuevo no genere conflicto con otro usuario;
3. valida la consistencia del valor ingresado.

Si el sistema muestra un mensaje indicando que un dato ya existe, revisa si otro usuario ya usa ese valor.

---

## Cómo importar usuarios desde CSV

1. Haz clic en `Importar CSV`.
2. Lee las instrucciones del cuadro de importación.
3. Selecciona el archivo CSV correcto.
4. Revisa la vista previa, si la pantalla la muestra.
5. Confirma la importación.
6. Espera a que finalice el proceso.
7. Revisa el resultado de la importación.

El sistema puede informar:

- usuarios creados;
- usuarios omitidos;
- errores de validación;
- conflictos de datos.

---

## Cuándo conviene usar la importación

Usa la importación cuando:

- debas cargar varios usuarios;
- estés incorporando personal nuevo en volumen;
- necesites actualizar datos desde una estructura previamente preparada.

Evita usar importación para corregir un solo caso aislado si puedes resolverlo por edición directa.

---

## Qué revisar antes de importar

1. confirma que el archivo sea realmente `.csv`;
2. revisa que el archivo tenga la estructura esperada;
3. valida que no existan filas incompletas;
4. confirma que no estás reimportando datos erróneos.

---

## Errores o situaciones frecuentes

### El usuario no aparece en la búsqueda

Revisa:

1. si el texto fue escrito correctamente;
2. si el usuario realmente fue cargado;
3. si el dato que usas para buscar coincide con el que está guardado.

### La importación falla

Revisa:

1. el formato del archivo;
2. columnas faltantes o datos inválidos;
3. duplicados;
4. mensajes de error devueltos por el sistema.

### Se importaron algunos usuarios y otros no

Eso significa que el sistema procesó parcialmente el archivo.

En ese caso:

1. revisa el detalle del resultado;
2. corrige solo las filas con error;
3. vuelve a importar el archivo corregido.

---

## Resultado esperado

Al finalizar, debes poder:

- encontrar rápidamente un usuario;
- corregir datos del sistema cuando corresponda;
- cargar usuarios de forma masiva con archivo CSV.

---

## Imágenes recomendadas

- pantalla principal del listado de usuarios;
- modal `Importar Usuarios`;
- formulario `Editar información del sistema`;
- ejemplo del resultado de una importación.
