# Proyecto1---IA-generativa

El sistema proyV3 implementa una arquitectura multiagente donde diferentes "especialistas" (agentes de IA) manejan tareas específicas. El objetivo es simular un entorno de trabajo real donde diferentes roles (médico, administrativo, coordinador) interactúan con una base de datos compartida (archivos Excel) para consultar información y registrar resultados.

## 1. Arquitectura y Componentes Clave

### Capa 1: Agentes (La Lógica de Decisión)
Sistema construido con **LangGraph** (`create_react_agent`). Se definen tres agentes con distintos permisos y prompts:

- **agente_coordinador**
  - **Rol:** Consulta general
  - **Permisos:** Buscar alumnos, ver estado de exámenes, generar reportes estadísticos
  - **Restricciones:** No puede registrar exámenes

- **agente_medico**
  - **Rol:** Entrada de datos clínicos
  - **Permisos:** Registrar exámenes, consultar alumnos
  - **Restricciones:** No puede generar reportes masivos

- **agente_administrativo**
  - **Rol:** Gestión masiva y reportería
  - **Permisos:** Listar alumnos pendientes, generar reportes
  - **Restricciones:** No puede registrar datos clínicos


###  Capa 2: Herramientas (Las Acciones)
Funciones de Python definidas con `@tool` de LangChain y validadas con Pydantic:

- `consultar_alumno(filtro_tipo, valor_filtro)`
  - **Descripción:** Busca alumnos en el Excel de ingresantes
  - **Filtros:** `"codigo"`, `"documento"`, `"apellido"`
  - **Salida:** Datos personales y progreso en exámenes

- `registrar_examen(codigo_matricula, tipo_examen, ...)`
  - **Descripción:** Registra o actualiza un examen
  - **Lógica:** Actualiza si existe, crea si no
  - **Uso:** Exclusivo del agente médico

- `obtener_estado_examenes(codigo_matricula)`
  - **Descripción:** Resumen detallado del progreso de un alumno

- `listar_alumnos_pendientes(tipo_examen, escuela)`
  - **Descripción:** Lista alumnos con exámenes pendientes
  - **Uso:** Principalmente para el agente administrativo

- `generar_reporte(tipo_reporte, parametro)`
  - **Descripción:** Estadísticas agregadas
  - **Tipos:** `"general"` o `"por_escuela"`
  - **Uso:** Coordinador y administrativo


###  Capa 3: Lógica de Datos (El "Backend")
Clase `GestorExcel` que actúa como un ORM para archivos Excel:

- **Propósito:** Abstraer lógica de lectura/escritura con pandas
- **Función:** Las herramientas no acceden directamente a los archivos, lo hace `GestorExcel`
- **Base de Datos:** Archivos `.xlsx` compartidos y persistentes

---

### Capa 4: Interfaz de Conversación (El "Frontend")
Funciones para interacción del usuario:

- `conversar(consulta, agente_tipo)`
- `cambiar_agente(nuevo_agente)`
- `SESION_ACTUAL`: Diccionario global que rastrea el agente activo y el `session_id`



## 2. Flujos Principales

### 🔍 Flujo de Consulta (Ejemplo: Buscar alumno por apellido)
```python
conversar("Buscar alumnos con apellido Rivera")
cambiar_agente("medico")
conversar("Registra laboratorio para 25010008, resultados normales, Dr. Pérez")

