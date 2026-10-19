# Cita-Studio
# Cita Studio

> Aplicación web para centralizar la gestión de citas de salones de belleza con múltiples sucursales.

## 1. Contexto del proyecto

Cita Studio es un proyecto académico de producto digital. Está diseñado para dueñas y personal de salones de belleza que operan dos o más sucursales y necesitan gestionar las citas de todos sus locales desde una agenda centralizada.

### Problema

La gestión de citas mediante agendas físicas y conversaciones dispersas dificulta la coordinación entre sucursales, aumenta el riesgo de errores y hace más complicado consultar las citas programadas.

### Propuesta de valor

Una sola agenda para saber qué citas están programadas, cuándo se atenderán y en qué sucursal.

## 2. Objetivo del MVP

Construir una aplicación web que permita al personal del salón:

- Iniciar sesión.
- Seleccionar una sucursal.
- Consultar las citas en una agenda diaria.
- Crear citas.
- Editar citas existentes.
- Cancelar citas sin eliminar permanentemente sus registros.
- Prevenir reservas que se solapen en la misma sucursal.
- Aplicar las reglas de fechas, duración y horario de atención.

El objetivo es entregar un flujo funcional y verificable, no una plataforma completa de gestión de salones.

## 3. Alcance funcional

### Pantallas

**1. Login**
- Campos de usuario y contraseña.
- Credenciales de demostración definidas para el prototipo.
- Navegación hacia la selección de sucursal tras un inicio de sesión válido.

**2. Selección de sucursal**
- Sucursales iniciales: Tuscania y Zaragoza.
- Botón «Agregar sucursal».
- La creación de nuevas sucursales queda fuera del MVP; el botón muestra «Funcionalidad en construcción».

**3. Agenda diaria**
- Muestra la fecha actual por defecto.
- Permite navegar al día anterior y al siguiente.
- Muestra las citas de la sucursal seleccionada para el día elegido.
- Horario visible: 8:00 a. m. a 6:00 p. m.
- Incluye el botón «Agregar cita».
- Permite seleccionar una cita para editarla o cancelarla.

**4. Formulario de cita**
- Nombre de la clienta.
- Fecha.
- Hora de inicio.
- Sucursal.
- Selección de uno a tres servicios.
- Duración calculada automáticamente.
- Acción principal para crear o guardar los cambios.

### Servicios disponibles

| Servicio | Duración |
|---|---:|
| Corte | 1 hora |
| Maquillaje | 1 hora |
| Secado o planchado | 1 hora |
| Tratamiento | 1 hora |

Cada cita dura la suma de las duraciones de los servicios seleccionados.

## 4. Reglas de negocio

Estas reglas son requisitos del producto y deben respetarse en todas las implementaciones relacionadas.

### Fechas permitidas

- Solo se pueden crear citas desde hoy hasta siete días después de hoy, ambos límites incluidos.
- Las fechas deben interpretarse según la fecha local del salón.
- Las citas pasadas no se pueden editar.
- La agenda puede permitir consultar fechas anteriores, pero no modificar citas pasadas.

### Horario de atención

- Apertura: 8:00 a. m.
- Cierre: 6:00 p. m.
- Todas las citas deben comenzar y terminar dentro de este intervalo.
- La hora de finalización no puede superar las 6:00 p. m.

Ejemplos:
- Una cita de dos horas puede comenzar a las 4:00 p. m.
- Una cita de dos horas no puede comenzar a las 5:00 p. m.
- Una cita de tres horas no puede comenzar a las 4:00 p. m.

### Prevención de doble booking

No pueden existir dos citas activas que se solapen en la misma fecha y sucursal.

La validación considera el intervalo completo de cada cita, no solo la hora de inicio.

Reglas adicionales:
- Las citas de sucursales diferentes pueden coincidir en horario.
- Dos citas consecutivas pueden compartir el límite entre el final de una y el inicio de otra.
- Al editar una cita, esta no debe generar un conflicto consigo misma.
- Una cita cancelada no debe bloquear el horario.
- Las validaciones importantes deben realizarse de forma segura en el servidor cuando exista almacenamiento compartido.

**Importante:** en el MVP, los conflictos se validan por sucursal. No se gestiona todavía la disponibilidad individual de las estilistas.

### Edición y cancelación

- Las citas de hoy o futuras pueden editarse, siempre que respeten las reglas de negocio.
- Las citas pasadas no pueden editarse.
- Cancelar una cita requiere confirmación.
- Una cita cancelada deja de mostrarse en la agenda activa.
- El registro debe conservarse internamente, por ejemplo, mediante un estado `cancelled`.
- Cancelar una cita libera su intervalo para nuevas reservas.
- No se deben eliminar físicamente los registros al cancelar.

## 5. Flujos principales

### Crear una cita

1. Iniciar sesión.
2. Seleccionar una sucursal.
3. Abrir la agenda diaria.
4. Pulsar «Agregar cita».
5. Completar los campos.
6. Calcular la duración total.
7. Validar fechas, horario y conflictos.
8. Guardar la cita.
9. Volver a la agenda de la fecha y sucursal correspondientes.
10. Mostrar la cita creada.

### Editar una cita

1. Seleccionar una cita en la agenda.
2. Elegir «Editar».
3. Mostrar el formulario con los datos actuales.
4. Modificar los campos.
5. Validar todas las reglas de negocio.
6. Guardar los cambios.
7. Volver a la agenda actualizada.

### Cancelar una cita

1. Seleccionar una cita.
2. Elegir «Cancelar».
3. Solicitar confirmación.
4. Marcar la cita como cancelada.
5. Ocultarla de la agenda activa.
6. Conservar el registro en el almacenamiento.

## 6. Datos y arquitectura

La aplicación debe diseñarse de forma que permita evolucionar sin tener que reescribir toda la lógica de negocio.

Entidades principales previstas:

### Sucursal

- `id`: identificador único.
- `name`: nombre de la sucursal.

### Cita

- `id`: identificador único.
- `client_name`: nombre de la clienta.
- `date`: fecha de la cita.
- `start_time`: hora de inicio.
- `end_time`: hora de finalización calculada.
- `branch_id`: referencia a la sucursal.
- `services`: servicios seleccionados.
- `duration_minutes`: duración total.
- `status`: estado de la cita, por ejemplo `scheduled` o `cancelled`.
- Fechas de creación y actualización, si la arquitectura lo requiere.

Estos campos representan el modelo lógico previsto. La estructura real debe verificarse en el código y en la base de datos antes de hacer cambios.

### Persistencia y usuarios compartidos

El objetivo del producto es que diferentes personas puedan consultar las mismas citas.

Si se utiliza una base de datos compartida, las citas deben persistir y ser accesibles desde diferentes sesiones autorizadas.

Los datos iniciales deben ser ficticios. No utilizar información personal real de clientas.

Si la implementación actual usa datos locales, memoria o simulaciones, no asumir que existe sincronización entre dispositivos. Identificar esa limitación antes de modificar la arquitectura.

## 7. Consideraciones futuras

Las siguientes funcionalidades están fuera del MVP actual:

- Selección de estilistas al crear una cita.
- Gestión administrativa del personal.
- Configuración de horarios laborales y disponibilidad individual.
- Portal público para que las clientas reserven sus propias citas.
- Consulta del historial de citas canceladas.
- Creación real de sucursales desde la interfaz.

No implementar estas funcionalidades salvo que el equipo decida explícitamente ampliar el alcance.

La arquitectura debe permitir que, en el futuro, los conflictos de disponibilidad consideren tanto la sucursal como la persona asignada.

## 8. Estado de implementación

Este documento define el comportamiento esperado del producto. No implica que todas las funciones estén implementadas.

Para conocer el estado real:
- Revisar el código actual.
- Comprobar las rutas y los componentes existentes.
- Verificar la conexión con el almacenamiento.
- Ejecutar la aplicación y probar los flujos disponibles.
- Identificar las diferencias entre los requisitos y la implementación.

Actualizar esta sección cuando existan cambios relevantes en el proyecto.

## 9. Guía para IAs que trabajen con este repositorio

Al utilizar una IA para desarrollar Cita Studio, seguir estas instrucciones:

1. Leer este README antes de proponer cambios.
2. Inspeccionar el código existente antes de editarlo.
3. Tratar el código como la fuente de verdad sobre lo que está implementado y este documento como la fuente de requisitos del producto.
4. No reconstruir ni reemplazar componentes que ya funcionen sin una razón concreta.
5. No inventar funcionalidades, campos, rutas, tablas ni dependencias.
6. Mantener el alcance del MVP y separar claramente las propuestas futuras.
7. Reutilizar los componentes, estilos y patrones existentes.
8. Centralizar las reglas de negocio para evitar validaciones inconsistentes.
9. No resolver la prevención de conflictos únicamente mediante validaciones visuales del frontend.
10. Ejecutar pruebas después de implementar o modificar una funcionalidad.
11. Probar los casos normales, los límites y los casos inválidos.
12. Corregir los errores detectados y volver a ejecutar las pruebas pertinentes.
13. Informar qué pruebas se ejecutaron, cuáles pasaron y cuáles no fue posible ejecutar.
14. No afirmar que una funcionalidad o prueba funciona sin haberlo verificado.
15. Si falta una decisión funcional importante, formular una pregunta antes de asumirla.

Antes de modificar archivos, explicar brevemente qué se va a cambiar cuando la modificación afecte a varias partes de la aplicación.

## 10. Herramientas y restricciones del proyecto

El proyecto debe desarrollarse priorizando herramientas gratuitas.

Lovable es la herramienta de construcción asistida por IA seleccionada inicialmente. No contratar Claude Code y Lovable simultáneamente.

GitHub se utiliza para almacenar el código, mantener el historial de cambios y facilitar la colaboración del equipo.

Cualquier servicio externo debe evaluarse según su disponibilidad gratuita, sus límites y las necesidades reales del MVP.

No incorporar dependencias pagadas sin autorización explícita del equipo.

## 11. Definición de terminado

Una funcionalidad no se considera terminada únicamente porque la interfaz se vea correcta.

Debe verificarse que:
- El flujo esperado funciona desde la interfaz.
- Se aplican las reglas de negocio correspondientes.
- Los datos se guardan y recuperan correctamente, cuando corresponda.
- Los errores se gestionan de forma comprensible.
- No se rompen los flujos existentes.
- Se ejecutaron las pruebas pertinentes o se documentaron las limitaciones.

El objetivo final es entregar un MVP funcional, consistente y verificable, respetando el alcance definido para Cita Studio.
