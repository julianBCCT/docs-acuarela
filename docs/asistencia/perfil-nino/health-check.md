---
sidebar_position: 4
---

# Health Check

Permite gestionar los Health Check diarios del niño. Su propósito es mostrar, añadir y visualizar registros médicos del niño por fecha a través de una vista calendario interactiva, todo lo relacionado a esta seccion se realiza en el archivo `healthcheck.js` y este a su vez se conecta a `modalhealth.js` para la creacion de un modal que permite ver y agregar reportes segun el dia seleccionado en el calendario.

Su flujo general es:
- Carga automática del día actual como seleccionado.
- Renderizado de calendario (mes, año, y días).
- Visualización del estado de salud diario.
- Selección de días pasados con datos para ver reportes.
- Posibilidad de agregar un nuevo reporte en días sin registro.


## Vista y creación del calendario

Este componente permite seleccionar cualquier día dentro de un calendario mensual dinámico. Además, filtra y muestra visualmente la información médica (`healthcheck`) del niño si hay datos para ese día.

```jsx
const [year, setYear] = useState(new Date().getFullYear());
const [month, setMonth] = useState(new Date().getMonth() + 1);
```

El año y mes seleccionable pueden ser cambiados o seleccionados por medio de dos modales, estos dias se generan dinámicamente:

```jsx
const years = Array.from({ length: 10 }, (_, i) => new Date().getFullYear() - i);
```

Es generado un array de los últimos 10 años para el selector de año, or medio de una lista de los 12 meses usando `translate.words` para internacionalización dinámica según el idioma.

```jsx
const months = [ ... ]; // Traducción dinámica de nombres de meses
```

El cálculo del calendario mensual devuelve el día de la semana (0–6) en el que cae el primer día del mes seleccionado, por ejemplo si el mes inicia un miércoles, firstDay = 3, calcula la cantidad de días del mes actual, el truco: al usar month, 0 se obtiene el último día del mes anterior, y por eso devuelve el total correcto del mes deseado, `Array(firstDay).fill(null)` añade espacios vacíos iniciales para alinear el calendario, `Array(totalDays).keys()` genera los días reales del mes (1 al totalDays), `.padStart(2, "0")`: asegura formato "01", "02", ..., "31".

```jsx
const firstDay = new Date(year, month - 1, 1).getDay();

const totalDays = new Date(year, month, 0).getDate();
 
const days = Array(firstDay).fill(null).concat(
  [...Array(totalDays).keys()].map(d => (d + 1).toString().padStart(2, "0"))
);
```

Para la interaccion de fefchas se da por medio de:
- `selectedDate`: define el día seleccionado por el usuario.
- `useEffect`: establece la fecha actual como default al cargar el componente.

```jsx
const [selectedDate, setSelectedDate] = useState("");
useEffect(() => {
  const today = new Date();
  const todayDate = `${today.getFullYear()}-${(today.getMonth() + 1).toString().padStart(2, "0")}-${today.getDate().toString().padStart(2, "0")}`;
  setSelectedDate(todayDate);
}, []);
```


Para el render de los dias se usa `FlatList` para mostrar los días del mes en una matriz de 7 columnas, cada día es `TouchableOpacity` (es decir permite una interaccion), y se colorea con base en 3 estados:
- Hoy → Color cielo.
- Tiene datos → Color fondo2.
- Seleccionado con datos → Color cielo desactivado.

```jsx
<FlatList
  data={days}
  numColumns={7}
  renderItem={({ item }) => {
    if (item === null) return <View style={styles.emptyBox} />;
    
    const dateString = `${year}-${month.toString().padStart(2, "0")}-${item}`;
    const isToday = dateString === selectedDate;
    const healthData = kid?.healthinfo?.healthcheck?.find(entry => entry.daily_fecha === dateString);
    const hasData = !!healthData;
    
    return (
      <TouchableOpacity
        onPress={() => {
          setSelectedDate(dateString);
          setSelectedHealthInfo({
            temperature: healthData?.temperature || "--",
            report: healthData?.report || "---",
            bodychild: healthData?.bodychild || "---"
          });
        }}
      >
        <Text>{item}</Text>
        <Text>{healthData?.temperature || "---"} °F</Text>
      </TouchableOpacity>
    );
  }}
/>
```
```jsx
const isToday = dateString === selectedDate;
const hasData = !!healthData;
```

Al seleccionar un día actualiza `selectedDate`, si existen datos ese día `setSelectedHealthInfo(...)` guarda: `temperature, report, bodychild`.



## Ver o agregar reporte

Dentro del flujo de la sección Daily Health Check, los botones permiten dos acciones mutuamente exclusivas dependiendo de si existe o no un reporte de salud para la fecha seleccionada, existen unas condiciones importantes de habilitación:
- `hasData`: determina si existe un `healthcheck` registrado para `selectedDate`.
- Botón "Ver reporte":
    - Solo está habilitado si `hasData === true`.
    - Usa `setviewModalVisible(true)` para mostrar el modal `ModalHealth` en modo lectura (mode="1").
- Botón "Agregar reporte":
    - Solo está habilitado si `hasData === false`.
    - Usa `setaddModalVisible(true)` para abrir el mismo modal pero en modo edición (mode="2").

La lógica detras del hasdata, busca si existe un `entry` para la `selectedDate` dentro del array `healthcheck`, si existe, se habilita el botón de "Ver reporte"; si no, el de "Agregar reporte".

```jsx
const selectedHealthData = kid?.healthinfo?.healthcheck?.find(
  (entry) => entry.daily_fecha === selectedDate
);
const hasData = !!selectedHealthData;
```

Por medio del modal reutilizable `ModalHealth` ambos botones reutilizan el mismo componente modal (ModalHealth) con diferente mode, mode="1" es para lectura solamente, mode="2" es un formulario editable para registrar datos nuevos, se muestran los valores de `selectedHealthInfo` si están presentes.

```jsx
<ModalHealth
  visible={viewmodalVisible}
  onClose={() => setviewModalVisible(false)}
  mode="1"
/>

<ModalHealth
  visible={addmodalVisible}
  onClose={() => {
    setaddModalVisible(false);
    setModalMode("2"); // resetear
  }}
  mode={modalMode}
  setModalMode={setModalMode}
/>
```

Cabe aclarar que este `ModalHealth` fue importado y su código de funcionamiento se encuentra en el archivo `app/(stack)/kidprofile/modalhealth.js`


## Modal para ver reporte
Cuando se abre el modal en `mode="1"`, se activa la visualización de un reporte de salud previamente registrado, esto permite al usuario (administrador o personal educativo) revisar información básica del niño correspondiente al día seleccionado.

Desde el componente HealthCheck se detecta si existe un healthcheck para la fecha (`hasData`), si existe, se habilita el botón "Ver reporte", al hacer clic:
```jsx
setviewModalVisible(true)
```
Se abre el ModalHealth con la prop mode="1".

Dentro de ModalHealth, si mode === "1" se renderiza esta sección:
- Se formatea la fecha (`date`) en formato legible para humanos (ej.: 23 de junio, 2025), esta fecha viene desde el componente padre y representa el día seleccionado en el calendario.
```jsx
<Text style={styles.fecha}>{date}</Text>
```

Para la temperatura, el valor numérico con unidad, por defecto "-- °F" si no hay datos, estos son recuperados desde `selectedHealthInfo.temperature`.
```jsx
<Text style={styles.text}>{`${temperature} °F`}</Text>
```

El reporte representa la nomenclatura del estado general (por ejemplo: OK, F-Feverish, etc.), se obtiene desde `selectedHealthInfo.report`.
```jsx
<Text style={styles.text}>{report}</Text>
```

Para la representación de la vista corporal del niño es una sección muy visual e importante que muestra dos ilustraciones del cuerpo humano (frontal y posterior), acompañadas de un círculo indicador si se reportó alguna parte específica del cuerpo afectada:
```jsx
<NinoHealthFront width={140} height={140} />
<NinoHealthBack width={140} height={140} />
```

El SVG del cuerpo frontal y posterior se carga como componentes visuales, si bodychild coincide con alguna parte registrada en bodyParts, se dibuja un círculo indicador con la posición específica usando:
```jsx
<Circle key={index} top={part.top} left={part.left} />
```

Coloca una marca visual exacta usando coordenadas top y left absolutas, mejora la representación visual de la zona afectada.
```jsx
const Circle = ({ label, top, left }) => (
  <View style={[styles.circleOverlay, { top, left }]}></View>
);
```


## Modal para agregar reporte

Este modo está diseñado para permitir al personal ingresar de manera rápida y estructurada el reporte de salud diario de un niño. Aparece al oprimir el botón "Agregar reporte" desde el componente `HealthCheck`, únicamente cuando no se ha ingresado un reporte para el día seleccionado, en este:
- Se verifica si hay datos (`hasData === false`) para la fecha seleccionada.
- Al oprimir "Agregar reporte", se abre el ModalHealth con mode="2".
- El usuario:
    - Visualiza la foto del niño y la fecha seleccionada.
    - Debe indicar si hay una novedad de salud (“¿Tienes alguna novedad con el control de salud diario?”).
    - Si responde “Sí”, se habilita un pequeño formulario para registrar:
        - Temperatura corporal.
        - Estado general de salud (nomenclatura).

Se muestra un bloque que contiene: Foto del niño (`kid.photo`), o una imagen por género si no hay foto, Nombre y fecha del día seleccionado (`kid.name` y `date`), esta visualización inicial ayuda a confirmar que el reporte se ingresará al niño correcto en la fecha indicada.

Para la pregunta respecto a la novedad si el usuario elige "No", el sistema asume automáticamente:
- Temperatura: "98"
- Estado de salud: "OK"
- No se muestra el formulario adicional.
Si selecciona "Sí", se habilita un conjunto de inputs adicionales como:
- Campo de texto para ingresar la temperatura registrada en °F.
- Un TouchableOpacity abre un modal con una lista (FlatList) de nomenclaturas como:
    - F-Feverish, D-Diarrhea, OK-Okay, etc.
- Al seleccionar una, se actualiza el estado formData.report y selectednomenclaturaValue.

```jsx
<TextInput value={formData.temperatura} />

<Modal visible={nomenclaturaVisible} />
```

El botón "Siguiente" solo se habilita si:
```jsx
novedad === "No" 
  || (novedad === "Si" && temperatura y report están completos)
```
Esto se controla mediante la variable isFormValid.

Este flujo permite registrar fácilmente si un niño tuvo una novedad de salud ese día, con la posibilidad de especificar detalles si aplica, mejora la trazabilidad y seguimiento del estado de los niños, vinculando la información con su perfil e historial.