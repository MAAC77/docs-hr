# 10.2 Estándar para Redacción de Manuales

---

## Objetivo

Definir un formato uniforme para los manuales de usuario del sistema.

Este estándar busca que todos los documentos:

- sean fáciles de leer
- se mantengan actualizados
- sirvan para capacitación y soporte

---

## Idioma

- Redactar en **español**
- Usar lenguaje operativo y directo
- Evitar términos internos de desarrollo cuando no aporten al usuario

---

## Estructura Mínima Obligatoria

Cada manual o capítulo de módulo debe incluir:

1. **Objetivo**
2. **Perfil al que aplica**
3. **Requisitos o permisos**
4. **Ruta o menú de acceso**
5. **Descripción de la pantalla**
6. **Procedimientos paso a paso**
7. **Reglas o validaciones visibles para el usuario**
8. **Errores frecuentes o advertencias**
9. **Resultado esperado**

---

## Plantilla de Encabezado

```md
# Nombre del Manual o Módulo

> Versión: 1.0
> Última actualización: YYYY-MM-DD
> Audiencia: Usuario Final | RRHH | Administrador
> Estado: Borrador | Vigente | Pendiente de revisión
```

---

## Reglas de Estilo

- Un procedimiento debe comenzar con un verbo: `Ingresar`, `Buscar`, `Registrar`, `Aprobar`, `Exportar`.
- Cada paso debe describir una sola acción.
- Si el flujo depende del rol, indicarlo antes del procedimiento.
- Si una pantalla tiene filtros, documentar:
  - qué filtros existen
  - cuándo usar cada uno
  - qué resultado producen
- Si una acción cambia datos, documentar el impacto esperado.

---

## Cuándo Separar un Manual

Crear un documento independiente cuando ocurra al menos una de estas condiciones:

- el módulo tiene más de 3 procesos principales
- el flujo tiene aprobaciones o validaciones complejas
- la pantalla es usada por varios roles con comportamientos distintos
- el módulo genera dudas frecuentes de soporte

---

## Convención de Archivos

Usar nombres estables y descriptivos:

- `manual-usuario-final.md`
- `manual-rrhh.md`
- `manual-administrador.md`
- `modulo-asistencia.md`
- `modulo-permisos.md`

Evitar nombres ambiguos como:

- `manual1.md`
- `nuevo-doc.md`
- `prueba.md`

---

## Mantenimiento

Actualizar un manual cuando cambie:

- la navegación
- el nombre de una pantalla
- un formulario
- una regla visible para el usuario
- un permiso por rol

La revisión mínima recomendada es por cada liberación funcional.
