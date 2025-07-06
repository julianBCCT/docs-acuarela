---
sidebar_position: 3
---

# Salud

La pestaña Salud permite visualizar y gestionar información médica del niño. Este módulo está diseñado para mostrar el historial de salud, los ungüentos autorizados, los datos del pediatra y ofrece un botón para agregar o actualizar información médica.

La lógica de esta sección se encuentra en el archivo:
`app/(stack)/kidprofile/saludnino.js`

## Estructura general del componente
La estructura base del componente está envuelta en un `<ScrollView>`, permitiendo desplazar el contenido si excede el alto de pantalla.

### Historial de Salud
Se despliegan diferentes campos del historial médico que provienen de kid.healthinfo, una propiedad cargada desde Strapi. Los campos incluyen:
- Alergias
- Asma
- Medicamentos
- Vacunas
- Accidentes
- Salud física
- Salud emocional
- Sospecha de abuso
Cada uno se muestra en bloques de la forma:

```jsx
<View style={styles.saludCampos}>
  <View style={styles.camposTitle}>
    <Image source={Checkbox} />
    <Text>Nombre del campo:</Text>
  </View>
  <View style={styles.camposData}>
    <Text>{valor extraído de `kid.healthinfo`}</Text>
  </View>
</View>
```
El componente valida si los campos son arrays y si tienen contenido. En caso contrario, se muestra "Ninguno" como valor por defecto.


### Ungüentos autorizados

Esta sección es plegable y muestra los ungüentos autorizados que se pueden aplicar al niño.
- El estado `isExpandedUnguentos` controla su visibilidad.
- Se usa un botón `<TouchableOpacity>` para expandir/colapsar el contenido.
- Se muestra un icono de flecha animado (`FlechaArriba`) mediante `rotateInterpolationUnguentos`.

```jsx
<TouchableOpacity onPress={toggleExpandUnguentos}>
  <Text>Ungüentos autorizados</Text>
  <Animated.View style={{ transform: [{ rotate: rotateInterpolationUnguentos }] }} />
</TouchableOpacity>

{isExpandedUnguentos && (
  <Text>{kid.healthinfo.ointments || "Ninguno"}</Text>
)}
```


### Datos del Pediatra
También es una sección expandible controlada por `isExpandedPediatra`.
Contiene:
- Nombre del pediatra
- Teléfono
- Correo electrónico

Se extraen desde `kid.healthinfo.pediatrician_*`. Si no hay datos, se muestra "Ninguno" por defecto.

```jsx
<TouchableOpacity onPress={toggleExpandPediatra}>
  <Text>Datos del pediatra</Text>
</TouchableOpacity>

{isExpandedPediatra && (
  <>
    <Text>Doctor: {kid.healthinfo.pediatrician}</Text>
    <Text>Teléfono: {kid.healthinfo.pediatrician_number}</Text>
    <Text>Email: {kid.healthinfo.pediatrician_email}</Text>
  </>
)}
```


### Sección de Incidentes de Salud
Esta sección permite visualizar un historial de eventos o incidencias médicas reportadas sobre el niño. Cada incidente puede contener datos como el tipo de incidente, descripción, gravedad, temperatura, estado de salud, y acciones tomadas y esperadas. También se incluye un botón para agregar nuevos reportes.
Ubicación del código: `app/(stack)/kidprofile/saludnino.js`

El componente itera sobre el array `kid.healthinfo.incidents`, que proviene del backend (Strapi), y renderiza cada incidencia como un bloque expandible.

```jsx
kid?.healthinfo?.incidents?.map((incidente, index) => (
  <View key={index}>
    <TouchableOpacity onPress={() => toggleExpandIncidente(index)}>
      ...
    </TouchableOpacity>
    {expandedIncidentes[index] && (
      <View>Contenido del incidente</View>
    )}
  </View>
))
```

Cada incidente se puede expandir/cerrar de forma individual usando un botón con una flecha (`AnimatedFlechaArriba`) que rota según el estado booleano `expandedIncidentes[index]`.
Este comportamiento permite al usuario explorar detalles solo del incidente que desea consultar.
Cada incidente desplegado contiene los siguientes campos:
- Número de incidente: `index + 1`	Se muestra como "Incidente X"
- Reportado por:	`incidente.reported_for`	Persona que registró la incidencia
- Fecha:	`incidente.reported_enf`	Fecha del evento, reformateada a DD-MM-YYYY
- Tipo de incidente:	`incidente.incident_type`	Tipo del evento registrado
- Descripción:	`incidente.description`	Detalles narrativos del incidente
- Gravedad:	`incidente.gravedad`	Nivel de severidad (leve, moderado, grave)
- Temperatura:	`incidente.temperature`	Valor numérico en °C
- Estado de salud: `incidente.statehealth`	Nomenclatura definida en el formulario de incidentes
- Acciones tomadas: `incidente.actions_taken`	Qué se hizo en respuesta al incidente
- Acciones esperadas: `incidente.actions_expected`	Recomendaciones o seguimiento esperado



##  Agregar datos de salud

Ubicado al final del componente, este botón permite al usuario ir a la vista de edición o creación de datos médicos (`agregarsalud.js`).
Este flujo permite:
- Crear información médica si no existe.
- Editar o actualizar información ya cargada.
- Redireccionar mediante el hook `router.push`.

```jsx
<TouchableOpacity onPress={toggleExpandPediatra}>
  <Text>Datos del pediatra</Text>
</TouchableOpacity>

{isExpandedPediatra && (
  <>
    <Text>Doctor: {kid.healthinfo.pediatrician}</Text>
    <Text>Teléfono: {kid.healthinfo.pediatrician_number}</Text>
    <Text>Email: {kid.healthinfo.pediatrician_email}</Text>
  </>
)}
```

### Agregar o Editar Datos de Salud `agreagarsalud.js`

El formulario accesible desde el botón "Agregar datos de salud" ubicado en la pestaña "Salud" de un niño, permite registrar por primera vez o actualizar la información médica general del menor.
Componente: `agregarsalud.js`

1. Carga inicial:
   - El `useEffect` detecta si ya existen datos médicos (`kid.healthinfo`) y los precarga en el formulario para su edición.
   - Si no hay información previa, los campos se inicializan vacíos.
   ```jsx
      // Para recibir los datos que ya se mandaron y colocarlos en los inputs al actualizar informacion
      useEffect(() => {
         if (kid?.healthinfo) {
            setFormData((prev) => ({
            ...prev,
            asthma: kid.healthinfo.asthma || "",
            allergies: Array.isArray(kid.healthinfo.allergies) && kid.healthinfo.allergies.length > 0
               ? [...kid.healthinfo.allergies]
               : [""],
            medicines: Array.isArray(kid.healthinfo.medicines) && kid.healthinfo.medicines.length > 0
               ? [...kid.healthinfo.medicines]
               : [""],
            vacination: Array.isArray(kid.healthinfo.vacination) && kid.healthinfo.vacination.length > 0
               ? [...kid.healthinfo.vacination]
               : [""],
            accidents: Array.isArray(kid.healthinfo.accidents) && kid.healthinfo.accidents.length > 0
               ? [...kid.healthinfo.accidents]
               : [""],
            physical_health: kid.healthinfo.physical_health || "",
            emotional_health: kid.healthinfo.emotional_health || "",
            suspected_abuse: kid.healthinfo.suspected_abuse || "",
            ointments: Array.isArray(kid.healthinfo.ointments) && kid.healthinfo.ointments.length > 0
               ? [...kid.healthinfo.ointments]
               : [""],
            pediatrician: kid.healthinfo.pediatrician || "",
            pediatrician_number: kid.healthinfo.pediatrician_number || "",
            pediatrician_email: kid.healthinfo.pediatrician_email || "",
            }));
         }
      }, [kid]);
   ```

2. Edición de campos:
   - Se permite ingresar alergias, medicamentos, vacunas, accidentes, condiciones físicas/emocionales, sospechas de abuso, ungüentos autorizados y datos del pediatra.

3. Guardar cambios:
   - Si hay datos previos → se actualiza la entrada existente en la colección `HEALTHINFOS` de Strapi.
   - Si no hay datos previos → se crea una nueva entrada.
   - La acción se ejecuta con `dispatch(createHealthInfo)` o `dispatch(editHealthInfo)` y redirige a `/kidProfile`.


Parte de la estructura y campos del formulario es la siguiente:
- `asthma`	Checkbox (1 o 0)	Indica si el niño tiene asma
- `allergies`	Array[String]	Lista dinámica de alergias ingresadas
- `medicines`	Array[String]	Lista dinámica de medicamentos
- `vacination`	Array[String]	Lista dinámica de vacunas recibidas
- `accidents`	Array[String]	Lista de eventos accidentales
- `physical_health`	String	Descripción del estado físico actual
- `emotional_health`	String	Descripción del estado emocional
- `suspected_abuse`	String	Cualquier sospecha de abuso a reportar
- `ointments`	Array[String]	Ungüentos, cremas, bloqueadores autorizados
- `pediatrician`	String	Nombre del médico tratante
- `pediatrician_number`	String	Teléfono de contacto del pediatra
- `pediatrician_email`	String	Correo electrónico del pediatra

Los campos de tipo array (`allergies`, `medicines`, `vacination`, `accidents`, `ointments`) permiten agregar múltiples elementos dinámicamente con un botón de "+".

```jsx
const handleAddField = (field) => {
  setFormData((prevData) => ({
    ...prevData,
    [field]: [...prevData[field], ""],
  }));
};
```

Para la validación del formulario, antes de guardar, se valida que los siguientes campos obligatorios no estén vacíos:
- Salud física (physical_health)
- Salud emocional (emotional_health)
- Sospecha de abuso (suspected_abuse)

En caso de que falte alguno, se muestra un Alert.alert() con los nombres de los campos faltantes traducidos.

```jsx
if (missingFields.length > 0) {
  Alert.alert("Campos incompletos", "Por favor completa los siguientes campos:\n• ...");
}
```

Por último, la lógica para guardar depende de si existe o no un `kid.healthinfo.id`:
- Si existe: se llama `editHealthInfo(id, data, kid.id)`
- Si no existe: se llama `createHealthInfo(data, kid.id)`

Después de guardar exitosamente:
- Se muestra un mensaje de confirmación.
- Se redirige a la pantalla principal de perfil (`/kidProfile`).



## Agregar nuevo reporte 

Al final de la sección se muestra un botón para crear un nuevo incidente:
- Este botón redirige a `agregarReporte.js`, donde el usuario completa un formulario.
- Los datos enviados se almacenan en Strapi y se reflejan automáticamente al volver a la sección de salud.
- El nuevo incidente se incluirá al principio del array `incidents`.

```jsx
<TouchableOpacity onPress={() => router.push("/kidProfile/agregarReporte")}>
  <Text>Agregar incidente</Text>
</TouchableOpacity>
```

###  Sección "Agregar nuevo reporte" `agregarReporte.js` 

Esta vista se activa al hacer clic en el botón "Agregar nuevo reporte" en la pestaña de Salud del perfil del niño, permite al personal registrar un incidente relevante ocurrido durante el día, el cual puede ser enviado al contacto de emergencia si así se desea.
Componente: `agregarReporte.js`
El flujo general se basa en: 
- Carga de datos del niño (kid) y traducciones (translate) desde Redux.
- Inicialización del formulario formData con campos vacíos.
- Interfaz de usuario para ingresar detalles del incidente.
- Validación obligatoria de campos claves.
- Envío a Strapi para almacenar el incidente.
- Opcional: Envío de correo al contacto de emergencia si el incidente es del día actual.

Dentro de `formData.incidents`, se recogen los siguientes valores:
- `reported_for`	Persona que reporta el incidente
- `incident_type`	Tipo de incidente (elegido desde un FlatList modal)
- `description`	Descripción detallada del evento
- `temperature`	Temperatura del niño si aplica
- `gravedad`	Nivel de gravedad (Leve, Moderado, Grave)
- `statehealth`	Estado de salud con nomenclatura (ej: F-Feverish)
- `actions_taken`	Qué medidas fueron tomadas en el momento
- `actions_expected`	Qué se espera que haga el acudiente o padre

La fecha se agrega automáticamente con `Intl.DateTimeFormat` al campo `reported_enf`.

Al oprimir "Guardar", se evalúa:
- Si el niño ya tiene datos de healthinfo:
   - Se agrega el nuevo incidente al arreglo incidents.
- Si no:
   - Se crea un nuevo objeto healthinfo con el primer incidente.

La acción se realiza con los métodos `dispatch(createHealthInfo)` o `dispatch(editHealthInfo)`.

Para el envío del reporte detallado, después de guardar el incidente, el sistema pregunta si se desea enviar un reporte detallado al contacto de emergencia (modal de "¿Deseas enviar?").
Si el usuario elige "Sí":
- Se valida si existe un incidente con la fecha actual (`todayFormatted`) en kid.`healthinfo.incidents`.
- Si existe, se usa `sendEmergencyEmail(...)` desde `emailService.js` para enviar el correo.

Si el usuario elige "No":
- Se redirige al perfil del niño.

Este flujo garantiza que solo se notifica al acudiente si hay un incidente del mismo día.