# IA Agente RRHH Bot - ChocolaTech 🤖🍫

Este proyecto es un agente de Inteligencia Artificial integrado con Telegram y Supabase diseñado para automatizar la gestión de consultas de Recursos Humanos (vacaciones, banco de horas, modalidad de trabajo) de la empresa **ChocolaTech**. 

El sistema cuenta con un **protocolo estricto de autenticación en dos pasos** basado en la identidad del chat de Telegram (`telegram_id`) y el Documento Nacional de Identidad (`dni`), garantizando la privacidad de los datos personales.

Puedes interactuar con el bot oficial en producción aquí: [AyudaInteligente_rh_bot](https://t.me/AyudaInteligente_rh_bot)

## 🚀 Características Clave
- **Autenticación Biométrica/Identitaria:** El bot verifica si el `telegram_id` existe en la base de datos antes de revelar información confidencial.
- **Auto-vínculo Seguro:** Si el usuario es nuevo, el agente le solicita su DNI y vincula automáticamente su cuenta de Telegram si el documento está libre.
- **Protección contra Usurpación:** El sistema impide que un tercero vincule un DNI que ya se encuentra activo en otra cuenta de Telegram.
- **Respuestas Híbridas:** Responde preguntas corporativas generales usando una Base de Conocimiento (RAG) o consultas de base de datos relacional (Supabase).

---

## 🛠️ Tecnologías y Nodos Utilizados (Stack)

- **n8n (Versión Avanzada de Agentes):** Orquestador del flujo y lógica de nodos de LangChain.
- **Cohere Chat Model:** Cerebro conversacional del agente de IA.
- **Cohere Embeddings (`embed-multilingual-v3.0`):** Modelo de vectorización de texto en múltiples idiomas.
- **Supabase Tool Nodes:** Herramientas personalizadas de lectura (`Get Many`) y escritura (`Update`) conectadas al Agente de IA.
- **LangChain Memory & Vector Store:** Buffer de ventana para retener el contexto de la conversación en base al ID del chat y almacén vectorial en memoria para documentación corporativa.
- **Telegram Trigger & Node:** Interfaz de entrada y salida para el usuario final.

---

## 🗄️ Arquitectura de la Base de Datos (Supabase)

Para recrear el entorno de datos e inicializar la base de datos independiente de este proyecto, ejecuta el siguiente script DDL directamente en el **SQL Editor** de tu panel de Supabase:

```sql
CREATE TABLE empleados (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    dni VARCHAR(8) UNIQUE NOT NULL,       
    telegram_id VARCHAR(50) DEFAULT NULL, 
    departamento VARCHAR(50) NOT NULL,
    puesto VARCHAR(100) NOT NULL,
    fecha_ingreso DATE NOT NULL,
    saldo_vacaciones INT NOT NULL DEFAULT 0,
    banco_horas NUMERIC(4, 1) NOT NULL DEFAULT 0.0,
    modalidad VARCHAR(20) NOT NULL
);

-- Registro de prueba sugerido para fases de QA en desarrollo
INSERT INTO empleados (nombre, email, dni, departamento, puesto, fecha_ingreso, saldo_vacaciones, banco_horas, modalidad)
VALUES ('Eric Monné', 'eric.monne@chocolatech.com', '87654321', 'Educación', 'Instructor de Cursos', '2025-01-15', 25, 12.5, 'Híbrido');
```
---
## 🔧 Instrucciones de Configuración en n8n
Sigue estos pasos detallados para importar y desplegar el agente en tu propia instancia de n8n:

### 1. Importar el Flujo
- Copia el archivo JSON completo de este proyecto.
- En tu panel de n8n, crea un flujo vacío (Create a workflow).
- Haz clic en los tres puntos de la esquina superior derecha y selecciona Import from File (o presiona Ctrl + V directamente sobre el lienzo).

### 2. Configurar Credenciales
Debes vincular tus cuentas en los respectivos nodos que muestran una alerta de conexión:
- Telegram API: Configura tu cuenta e introduce el Token de Bot obtenido mediante @BotFather.
- Cohere API: Genera tu API Key en la plataforma de Cohere y asígnala a los nodos Cohere Chat Model y Embeddings Cohere.
- Supabase API: Ve a Project Settings > API en Supabase. Copia la Project URL y el service_role Secret Key (necesario para otorgar permisos de escritura en el nodo de actualización). Asígnala a los dos nodos de Supabase del flujo.

### 3. Sincronizar Evento de Prueba
Abre el nodo Telegram Trigger y haz clic en Listen for test event or simplemente ejecuta el workflow una vez. Luego, ve e a Telegram y envía un mensaje ("Hola") a tu bot. Esto cargará la metadata real del chat en el lienzo, eliminando las alertas de campos indefinidos (undefined).

### 4. Publicar
Cambia el nombre del flujo a tu preferencia y cambia el interruptor de la esquina superior derecha a ACTIVE.

---

## 👥 Flujo de Conversación (Prueba de Concepto)
- Intento de Acceso: El usuario pregunta "¿Cuántas vacaciones tengo?". El bot detecta que no hay un telegram_id enlazado y frena la respuesta exigiendo el DNI.
- Registro Exitoso: El usuario introduce el DNI de prueba (87654321). El bot valida que la columna telegram_id está libre (NULL), vincula la cuenta de Telegram y responde con éxito.
- Persistencia: Al volver a preguntar por datos personales, el bot responde directamente sin pedir identificaciones adicionales.
