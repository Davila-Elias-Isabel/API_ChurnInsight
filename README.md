<div align="center">
  <img
    src="https://github.com/user-attachments/assets/cb3f82c7-9cbc-4a10-ab12-1efc4e9a5828"
    alt="Churn Insight Logo"
    width="200"
  />
</div>

<h1 align="center">📊 Churn Insight</h1>
<h3 align="center">Plataforma Web de Análisis y Predicción de Cancelación (Customer Churn)</h3>

<hr/>

## 🧠 Descripción

**Churn Insight** es una plataforma web que permite:

- **Predicción manual** del riesgo de churn (formulario).
- **Búsqueda por ID (publicId)** para consultar un cliente existente y su probabilidad de churn.
- **Dashboard (Análisis Avanzado)** con gráficas interactivas basadas en estadísticas del modelo.
- **Exportación a PDF** (1 gráfico por página con logo y títulos).

Este repositorio incluye **Frontend (HTML/CSS/JS)** + **Backend Spring Boot** que funciona como **API Gateway** hacia un microservicio externo de Machine Learning (FastAPI).

---

## 📌 Tabla de Contenido

- [Arquitectura](#-arquitectura)
- [Tecnologías](#-tecnologías)
- [Requisitos](#-requisitos)
- [Ejecución local](#-ejecución-local)
- [Ejecución con Docker](#-ejecución-con-docker)
- [Configuración del microservicio ML](#-configuración-del-microservicio-ml)
- [Swagger / OpenAPI](#-swagger--openapi)
- [API Endpoints](#-api-endpoints)
- [Frontend](#-frontend)
- [Exportación PDF](#-exportación-pdf)
- [Errores y troubleshooting](#-errores-y-troubleshooting)
- [Backlog / Mejoras sugeridas](#-backlog--mejoras-sugeridas)
- [Equipo](#-equipo)

---

## 🏗️ Arquitectura

```
[ Frontend Web (Thymeleaf + static: HTML/CSS/JS + Chart.js + jsPDF) ]
                         ↓
[ Backend Spring Boot (API Gateway / Normalización / Manejo de errores) ]
                         ↓
[ Microservicio ML Externo (FastAPI) ]
    - Predicción manual
    - Predicción por ID
    - Endpoints de probability (stats)
```

> Nota importante: en este momento las URLs del microservicio externo están **hardcodeadas** en el código Java (ver sección de configuración).

---

## 🧰 Tecnologías

**Backend**
- Java 17
- Spring Boot 3.5.8
- Spring Web / Thymeleaf
- Lombok
- Springdoc OpenAPI (Swagger UI)
- RestTemplate (llamadas HTTP al microservicio ML)

**Frontend**
- HTML / CSS / JavaScript
- Chart.js (gráficas)
- jsPDF (exportación a PDF)

---

## ✅ Requisitos

- **Java 17**
- (Opcional) **Maven** — el proyecto incluye **Maven Wrapper** (`mvnw`, `mvnw.cmd`)
- Acceso al **microservicio ML externo** (si no está accesible, la API de predicción/stats fallará)

---

## ▶️ Ejecución local

### Linux / macOS
```bash
./mvnw spring-boot:run
```

### Windows (PowerShell / CMD)
```bat
mvnw.cmd spring-boot:run
```

Una vez arriba:
- **Frontend**: `http://localhost:8080/`
- **API base**: `http://localhost:8080/`
- **Swagger UI**: `http://localhost:8080/swagger-ui/index.html`

---

## 🐳 Ejecución con Docker

> El `Dockerfile` asume que el JAR ya existe en `target/`. Primero hay que compilar.

1) Construir el JAR
```bash
./mvnw clean package
```

2) Build de la imagen
```bash
docker build -t churninsight-api .
```

3) Run del contenedor
```bash
docker run --rm -p 8080:8080 churninsight-api
```

---

## 🔧 Configuración del microservicio ML

Actualmente hay **3 endpoints externos** definidos directamente en el código:

### 1) Predicción manual (FastAPI)
Archivo: `src/main/java/com/churninsight/api/service/PredictionService.java`
```java
private static final String MODEL_PREDICT_URL = "http://168.197.48.239:8000/predict";
```

### 2) Predicción por ID (FastAPI)
Archivo: `src/main/java/com/churninsight/api/service/PredictionService.java`
```java
private static final String MODEL_ID_URL = "http://168.197.48.239:8000/item/predictions/";
```

### 3) Estadísticas probability (Cloudflare tunnel)
Archivo: `src/main/java/com/churninsight/api/service/StatsService.java`
```java
private static final String BASE_URL = "https://definitely-poetry-few-bachelor.trycloudflare.com";
```

📌 **Si cambian URLs/túneles**, se deben actualizar estas constantes.

> Recomendación (mejora futura): mover estas URLs a `application.properties` y/o variables de entorno para no recompilar.

---

## 📘 Swagger / OpenAPI

El proyecto incluye Swagger UI vía `springdoc-openapi-starter-webmvc-ui`.

- Swagger UI:  
  `http://localhost:8080/swagger-ui/index.html`

- OpenAPI JSON:  
  `http://localhost:8080/v3/api-docs`

---

## 🔌 API Endpoints

### 1) Predicción manual
**POST** `/predict`

**Request (JSON)**  
> Importante: los campos están en **snake_case** (ej. `subscription_type`, `watch_hours`).

```json
{
  "age": 52,
  "gender": "Male",
  "subscription_type": "Premium",
  "watch_hours": 1.1,
  "region": "Europe",
  "number_of_profiles": 3,
  "payment_method": "credit card",
  "device": "tv"
}
```

**Response (ejemplo)**
```json
{
  "prediction": 1,
  "probabilities": {
    "churn": 0.83,
    "not_churn": 0.17
  }
}
```

---

### 2) Predicción por ID (publicId)
**GET** `/predict/client/{publicId}`

**Response (ejemplo)**
```json
{
  "prediction": 1,
  "probability": 0.83,
  "client": {
    "age": 52,
    "gender": "Male",
    "subscription_type": "Premium",
    "watch_hours": 1.1,
    "region": "Europe",
    "number_of_profiles": 3,
    "payment_method": "credit card",
    "device": "tv"
  }
}
```

✅ **Errores 404 con detalle**
Si el microservicio externo devuelve `{"detail":"..."}`, el backend lo normaliza y responde con:
```json
{
  "detail": "mensaje",
  "message": "mensaje"
}
```

---

### 3) Estadísticas (Análisis Avanzado)
**GET** `/probability/gender`  
**GET** `/probability/region`  
**GET** `/probability/subscription`  
**GET** `/probability/age`

**Response esperada (ejemplo)**
```json
{
  "totalUsers": 0,
  "data": [
    {
      "label": "Male",
      "churnProbability": 34.2,
      "notChurnProbability": 65.8,
      "usersCount": 120
    }
  ]
}
```

📌 Nota: `StatsService` actualmente devuelve un `StatsResponseDTO` vacío si ocurre cualquier error al consumir el servicio externo (comportamiento “silencioso”).

---

## 🖥️ Frontend

El frontend se sirve desde:
- `GET /` → renderiza `templates/index.html`

Secciones (tabs):
- **Cálculo Manual**: formulario → `POST /predict`
- **Búsqueda**: por publicId → `GET /predict/client/{id}`
- **Análisis Avanzado**: stats + charts → `GET /probability/*`

⚠️ Importante para despliegue:
En `static/js/app.js` las llamadas `fetch()` están en **URL absoluta**:
- `http://localhost:8080/...`

Si se despliega en otro host/dominio, se recomienda cambiar a rutas relativas:
- `/predict`
- `/probability/gender`
- etc.

---

## 🖨️ Exportación PDF

En **Análisis Avanzado** existe el botón **Exportar a PDF**:
- Exporta **1 gráfico por página**
- Incluye logo (`/img/logo.png`) y títulos
- Nombra el PDF con timestamp (HHMMSS)

---

## 🧯 Errores y troubleshooting

### 1) “No carga /predict o /probability”
- Verifica que el backend esté arriba en `http://localhost:8080/`
- Verifica conectividad con el microservicio externo:
  - `MODEL_PREDICT_URL` / `MODEL_ID_URL`
  - `BASE_URL` (Cloudflare)

### 2) 404 al buscar cliente por ID
- El backend responde con `detail` y `message` si el servicio externo devuelve error.

### 3) Dashboard vacío
- `StatsService` devuelve respuesta vacía si hay error en la respuesta o formato inesperado del JSON (por ejemplo, si `data` no es lista o faltan campos).

---

## 🧩 Backlog / Mejoras sugeridas

1) **Parametrizar URLs externas** (properties/env) en vez de hardcode.
2) Cambiar `fetch("http://localhost:8080/...")` a rutas relativas para despliegue.
3) Robustecer `StatsService`:
   - Validar `nulls`
   - Manejar tipos inesperados sin silenciar errores (log + respuesta informativa)
4) Añadir **validaciones Bean Validation** a `ModelDataDTO` (`@NotNull`, rangos, etc.) y tests.
5) Unificar consistencia de valores del formulario vs valores del modelo (case sensitive).
6) Mejorar observabilidad: logs controlados y trazabilidad de fallos del servicio externo.

---

## 👥 Equipo DracoStack

- **Hernán Cerda** - Backend Developer
- **Silvia Hernández** - Backend Developer
- **Aldo Sánchez** - Backend Developer
- **Kenny Solórzano** - Backend Developer
- **Leslie Rodriguez** - Data Engineer
- **Rocio Isabel Davila** - Data Scientist
- **Elizabeth Garces** - Data Scientist

<hr/>

<div align="center">
  <p><i>Churn Insight — Integración práctica entre Spring Boot y Machine Learning.</i></p>
</div>
