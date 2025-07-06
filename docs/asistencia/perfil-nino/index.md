---
sidebar_position: 1
---

# Perfil del Niño

Este apartado documenta el comportamiento y estructura visual del perfil de un niño dentro de la app, así como la lógica de navegación y componentes principales involucrados.


## Estructura de Carpetas para perfil del niño

```bash
.
├── app/                          # Enrutamiento con Expo Router (app directory routing)
│   ├── (stack)/                  # Rutas internas agrupadas en un stack
│   ├── auth/                     # Pantallas de autenticación (login, registro, etc.)
│   ├── kidprofile/               # Carpeta de ubicacion de todos de todo el contenido interno en el perfil de un niño. 
│       ├── utils/
│           ├── emailService.js   
│       ├── layout.js
│       ├── agregarReporte.js
│       ├── agregarsalud.js
│       ├── healthcheck.js
│       ├── index.js
│       ├── modalemergencia.js
│       ├── modalhealth.js
│       ├── saludnino.js
│   ├── mensajes/ 
# etc.)                            
│   ├── auth/               # Pantallas de autenticación (login, registro,   
│   └── index.js                      
```

## Estructura visual inicial

El diseño del perfil de un niño se realiza en (`kidProfile/index.js`), podemos observar en la app la siguiente estructura visual:

- **Nombre completo del niño** (encabezado)
- A la izquierda:
  - 📅 Fecha de nacimiento
  - 📆 Inscrito desde
  - 🏙 Ciudad
  - 📞 Número mamá
  - 📧 Correo mamá
  - 📞 Número papá
  - 📧 Correo papá
- A la derecha:
  - Foto de perfil del niño
  - Botón de "Contacto de emergencia"


Los íconos utilizados en este apartado están definidos como componentes React dentro de archivos .js que contienen código SVG (formato vectorial escalable) y se ubican en assets/icons/.

Estos íconos se importan e instancian como componentes en el archivo index.js.

```jsx
import Evento from "../../../assets/icons/evento";
import Calendario from "../../../assets/icons/calendario";
import Localizacion from "../../../assets/icons/localizacion";
import Telefono from "../../../assets/icons/telefono";
import Mensajes from "../../../assets/icons/mensajes";
```

Luego para usarlas se instancia el componente SVG y se le pasan propiedades `props` como si fuera un componente normal, para este apartado los iconos tienen dimensiones de `width={16}` y `height={16}`, esto se traduce en que el archivo SVG contiene vectores que son renderizados directamente como elementos gráficos nativos via (`react-native-svg`), lo que permite que se rendericen de forma nativa en dispositivos móviles.

```jsx
<Evento width={16} height={16} style={[styles.icon]} />
```

Cada ítem de información sigue una estructura similar: un contenedor View contenidos en una clase llamada `rowDataContainer` donde va el ícono y el texto (traducible), seguido del dato dinámico proveniente de la API de Strapi. Por ejemplo, el campo de Fecha de nacimiento:

```jsx
<View style={styles.rowDataContainer}>
  <Evento width={16} height={16} style={[styles.icon]} />
  <Text style={styles.titleData}>
    {
      translate?.words?.find((word) => word.id == 195)[
        translate.language
      ]
    }
  </Text>
</View>
<Text style={styles.textData}>
  {moment(kid ? kid.birthday : "").format("MM-DD-YYYY")}
</Text>
```
El componente `translate?.words?.find(...).language` permite que los textos puedan mostrarse dinámicamente en distintos idiomas (por defecto español o inglés) dependiendo de la configuración del usuario.


### Botón CONTACTO DE EMERGENCIAS

Para crear el boton de emergencia se utiliza el componente personalizado `MainButton`, al ser presionado, abre un modal que permite contactar de forma inmediata a números de emergencia o enviar alertas a contactos registrados.

```jsx
<MainButton
  style={styles.contactEmergency}
  buttonTxt={
    translate?.words?.find((word) => word.id == 203)?.[
      translate.language
    ]
  }
  primario={true}
  onPress={() => setModalEmergencyVisible(true)}
  hasIcon={false}
/>
```

El componente del modal se encuentra en (`kidProfile/modalemergencias.js`), para implementarlo aca primero se importa:

```jsx
import ModalEmergencia from "./modalemergencia";
```

Luego se declara un estado para controlar su visibilidad usando el hook `useState` de React, useState es un hook de React que permite definir variables de estado locales dentro de un componente funcional. En este caso, `modalemergencyVisible` determina si el modal está visible (`true`) u oculto (`false`).

```jsx
const [modalemergencyVisible, setModalEmergencyVisible] = useState(false);
```

Finalmente, el componente `ModalEmergencia` se renderiza y recibe las siguientes props, aquí, permiten que el modal tenga acceso a los datos del niño y sus contactos para mostrar la información adecuada y ejecutar acciones como llamadas o envío de correos automáticos.

```jsx
{/* Modal de Emergencias */}
<ModalEmergencia
  visible={modalemergencyVisible}
  onClose={() => setModalEmergencyVisible(false)}
  kid={kid}
  parents={kid?.acuarelausers || []}
  guardians={kid?.guardians || []}
/>
```


#### Estructura en `kidProfile/modalemergencias.js`
Estados principales:
- `mode`: Controla cuál de los 3 submodales se muestra (inicial: "1").
- `floatingButtonsVisible`: Muestra opciones de "llamar" y "email" en caso urgente.
- `floatingButtons2Visible`: Muestra la opción de “llenar reporte” si aún no existe uno para el día.
- `message`: Muestra avisos al usuario.
- `visible`: Muestra u oculta el modal completo.

Estructura por niveles de modal:

```jsx
Botón Contacto de Emergencias
Nivel 1 → Llamar a Emergencias
   ├── Nivel 2 → Urgencias 911
   └── Nivel 2 → Policia

Nivel 1 → Contactar pariente
   ├── Nivel 2 → Caso Urgente
   |           ├── Nivel 3 → Botón flotante llamar a contacto de emergencia
   |           └── Nivel 3 → Botón enviar email urgente a contacto de emergencia   
   └── Nivel 2 → Enviar reporte detallado
   |           ├── Nivel 3 → Botón flotante en caso de no existir reporte ese dia para ese niño.
   |           └── Nivel 3 → Envia email detallado en caso de que si se halla registrado reporte ese día.
```

El primer modal se define con un if si `mode === "1"`este contendra el título del modal junto con 2 botones definidos en código por el componente `TouchableOpacity`.
- "Llamar a emergencias" → cambia mode a "2".
- "Contactar pariente" → cambia mode a "3".
También se visualiza la sección `EmergencyContacts`, que muestra los datos del contacto de emergencia disponibles desde Strapi (guardianes[]).

El submodal presenta 2 opciones, cada botón ejecuta su función correspondiente (urgency911, policy) usando `Linking.openURL("tel:...")`, lo que abre el marcador telefónico del dispositivo, por ejemplo para el 911 se hace con la siguiente función.

```jsx
const urgency911 = () => {
  const phoneNumber = "tel:911";
  Linking.canOpenURL(phoneNumber)
    .then((supported) => {
      if (!supported) {
        Alert.alert("No se puede abrir el marcador telefónico");
      } else {
        Linking.openURL(phoneNumber);
        if (Platform.OS === "ios") {
          Alert.alert(
            "Llamando",
            "Confirma la llamada desde la app de Teléfono."
          );
        }
      }
    })
  .catch((err) => console.error("Error al intentar llamar", err));
};
```

Para el botón "Contactar pariente" existen otros 2 botones que realizan ciertas subacciones:
A. Caso Urgente
Al presionar este botón, se activa `setFloatingButtonsVisible(true)`, aparecen dos botones flotantes:
- Llamar: usa el teléfono del primer contacto disponible (`guardians[0]` o `parents[0]`) con validación del número (`handleCall()`).
- Email: se envía un correo automático urgente con sendEmergencyEmail("grave", ...) desde emailService.js.

B. Enviar reporte detallado
Este botón evalúa si ya existe un incidente para hoy:
- Si existe: se envía un correo detallado con datos como temperatura, tipo de incidente, acciones tomadas, etc.
- Si no existe: se muestra un botón flotante "Llenar reporte", que redirige al componente `agregarReporte.js` con `router.push("/kidProfile/agregarReporte")`.

El envío de correos con 'emailService.js', este archivo se encarga de las funciones de correo:
- `sendEmergencyEmail(...)`: usado en "Caso urgente" y "Reporte detallado"
- Soporta distintos niveles de gravedad: "grave" y "moderado"
Se invoca con los datos extraídos dinámicamente de los contactos asociados y los incidentes registrados en Strapi.

## Tabs disponibles

El perfil del niño incluye una sección con pestañas (tabs) que permiten navegar entre diferentes apartados relacionados con su información. Estos tabs están implementados de forma dinámica usando el hook useState, componentes condicionales, y el renderizado de contenido basado en el valor de una variable de estado.


Primero se importan los componentes individuales que representan cada sección (tab), como Salud y HealthCheck, estos componentes encapsulan la lógica, renderizado y estilos de cada apartado, como, SaludNino muestra el historial médico, y HealthCheck permite ver o crear reportes diarios de salud.

```jsx
import HealthCheck from "./healthcheck";
import SaludNino from "./saludnino";
```


### Estado activo del tab

Se declara un estado llamado `activeTab` usando `useState`, el cual mantiene el tab actualmente activo (por defecto, `"familia"`):

- `activeTab`: representa el id del tab actualmente seleccionado.
- `setActiveTab`: función que actualiza el valor de `activeTab`.

```jsx
const [activeTab, setActiveTab] = useState("familia");
```

Cada tab tiene:
- `id`: usado como identificador interno.
- `wordId`: clave de traducción para mostrar el nombre del tab en varios idiomas.

```jsx
const tabs = [
  { id: "familia", wordId: 204 },
  { id: "salud", wordId: 205 },
  { id: "health_check", wordId: 206 },
  { id: "actividades", wordId: 207 },
  { id: "Pagos", wordId: 208 },
  { id: "Adjuntos", wordId: 209 },
];
```


### Renderizado de la barra de navegación

El componente visual que representa la barra de tabs se genera dinámicamente a partir del array `tabs`, usando `map()`:

-`TouchableOpacity`: crea un botón táctil por cada tab.
-Al hacer `onPress`, se llama a `setActiveTab(tab.id)`, lo cual cambia la pestaña activa.
-El `label` se traduce dinámicamente mediante `translate.words`.

```jsx
<View style={styles.navtabs}>
  {tabs.map((tab) => {
    const label =
      translate?.words?.find((word) => word.id === tab.wordId)?.[
        translate.language
      ] || tab.id;

    return (
      <TouchableOpacity
        key={tab.id}
        style={[
          styles.navtab,
          activeTab === tab.id && { color: Colors.cielo.normal },
        ]}
        onPress={() => setActiveTab(tab.id)}
      >
        <Text
          style={[
            styles.tabText,
            activeTab === tab.id && styles.activeText,
          ]}
        >
          {label}
        </Text>
      </TouchableOpacity>
    );
  })}
</View>
```

### Renderizado de la barra de navegación

Según el valor de `activeTab`, se renderiza condicionalmente el contenido de la sección correspondiente.

- `FlatList` se usa para mostrar listas horizontales como los contactos (acuarelausers) o los archivos adjuntos (`files`).
- En `"familia"` se filtran los usuarios con rol específico (ID de cuidador o representante).
- En `"Adjuntos"` se usa el componente `FileCard` para representar cada archivo.
- En `"salud"` y "health_check" se montan componentes completos (SaludNino, HealthCheck), que tienen su propia lógica de estado y vista.

```jsx
<View style={[styles.container, { flex: 1 }]}>
  {activeTab === "familia" && (
    <View style={{ flexDirection: "column" }}>
      <FlatList ... /> {/* Mostrar contactos familiares */}
    </View>
  )}
  {activeTab === "actividades" && <View></View>}
  {activeTab === "Pagos" && <View></View>}
  {activeTab === "Adjuntos" && (
    <FlatList
      fadingEdgeLength={30}
      data={kid.files}
      renderItem={({ item, index }) => (
        <FileCard
          key={index}
          style={{ margin: 0, marginBottom: 15 }}
          typeFile={1}
          titleCard={item.name}
        />
      )}
    />
  )}
  {activeTab === "health_check" && <HealthCheck />}
  {activeTab === "salud" && <SaludNino />}
</View>
```