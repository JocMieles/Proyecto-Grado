# Proyecto-Grado
Proyecto de Grado UNAD

# Descripción del proyecto

Este proyecto implementa un prototipo de automatización de PQRS (Peticiones, Quejas, Reclamos, Solicitudes y Felicitaciones) para una empresa de servicios en Colombia usando n8n, OpenAI y Google Sheets.

El flujo permite:
	•	Recibir PQRS desde:
	•	Correos electrónicos (Gmail Trigger).
	•	Un Webhook (por ejemplo, formularios web o pruebas con Postman).
	•	Normalizar los datos de entrada (nombre, correo, canal, asunto, mensaje).
	•	Consultar el historial de PQRS del usuario en una hoja de cálculo.
	•	Usar un AI Agent para:
	•	Clasificar el tipo de PQRS.
	•	Asignar prioridad y área responsable.
	•	Evaluar si faltan datos del usuario (cédula, medicamento, orden médica, etc.).
	•	Generar un resumen del caso.
	•	Redactar una respuesta inicial para el usuario.
	•	Definir y actualizar el estado de la PQRS (nuevo, pendiente_datos, en_proceso, resuelto, cerrado).
	•	Registrar o actualizar automáticamente el caso en Google Sheets.
	•	Responder al usuario por correo con la información generada por la IA.

Este prototipo integra servicios reales (Gmail, Google Sheets, API de OpenAI y n8n) y sirve como demostración de un sistema con nivel de madurez tecnológica cercano a TRL 5.

# Diagrama simple del flujo

A nivel general, el flujo en n8n sigue estos pasos:
	1.	Entrada
	•	Gmail PQRS (Gmail Trigger): captura nuevos correos recibidos en el buzón de PQRS.
	•	Webhook: permite recibir PQRS desde aplicaciones externas o Postman.
	2.	Normalización
	•	Normalizar Email / Normalizar Webhook: extraen y unifican los campos nombre, correo, canal, asunto, mensaje.
	3.	Consulta de historial
	•	Get row(s) in sheet: busca en Google Sheets si ya existe una PQRS previa para ese correo.
	4.	Preparación para IA
	•	Normalizar Email2: combina datos del mensaje y datos de la hoja (estado actual, etc.) en un solo objeto.
	5.	Agente de IA
	•	AI Agent:
	•	Llama al modelo de OpenAI con un prompt especializado en PQRS.
	•	Usa Simple Memory para contexto de conversaciones.
	•	Usa la tool pqrs (Google Sheets) para consultar/actualizar el registro del caso.
	•	Devuelve un JSON con tipo, prioridad, área, resumen, respuesta, estado_pqrs, datos_faltantes y registro_pqrs.
	6.	Procesamiento de salida
	•	Procesar respuesta (Function/Code): adapta el JSON de la IA al formato requerido por:
	•	la hoja de cálculo (columnas), y
	•	el correo de respuesta.
	7.	Persistencia y respuesta
	•	Append or update row in sheet: crea o actualiza la fila correspondiente en Google Sheets.
	•	Send a message (Gmail): envía la respuesta generada al usuario.

# Requisitos

Para ejecutar el workflow se necesitan:

Entorno
	•	Una instancia de n8n (Cloud, Docker o Desktop).
	•	Acceso a internet desde n8n para llamar a la API de OpenAI y a los servicios de Google.

Credenciales y servicios externos
	1.	OpenAI
	•	Cuenta de OpenAI.
	•	API Key válida.
	•	Credencial configurada en n8n (por ejemplo OpenAI Chat Model o similar).
	2.	Google (Gmail y Sheets)
	•	Cuenta de Google con acceso a Gmail y Google Sheets.
	•	Credencial de Google en n8n (OAuth2 o Service Account) con permisos para:
	•	leer correos del buzón de PQRS (Gmail),
	•	leer y escribir en la hoja de cálculo usada como base de datos de PQRS.
	•	Una hoja de Google Sheets creada previamente, con al menos estas columnas (puedes ajustar nombres):
	•	correo
	•	nombre
	•	canal
	•	fecha
	•	tipo
	•	prioridad
	•	area
	•	estado_pqrs
	•	resumen
	•	(opcionales) datos_faltantes, palabras_clave, ultima_respuesta, etc.
	3.	Otros (opcional para pruebas)
	•	Postman u otra herramienta para enviar solicitudes al Webhook de n8n.
	•	Cuenta de correo de prueba para enviar y recibir PQRS.

# Pasos para importar el workflow en n8n
	1.	Obtener el archivo del workflow
	•	Descarga o guarda el archivo JSON del flujo (por ejemplo workflow-pqrs-n8n.json).
	2.	Crear el workflow en n8n
	•	Ingresa a tu instancia de n8n.
	•	Haz clic en Workflows → Import from File (o el botón de importar).
	•	Selecciona el archivo workflow-pqrs-n8n.json.
	•	Guarda el workflow con un nombre descriptivo (por ejemplo, “PQRS IA – n8n”).
	3.	Configurar credenciales
	•	Abre cada nodo que requiere credenciales y asígnales la adecuada:
	•	Gmail PQRS y Send a message: selecciona tu credencial de Gmail/Google.
	•	Get row(s) in sheet y Append or update row in sheet (tool pqrs): selecciona la credencial de Google Sheets y el documento/hoja correctos.
	•	OpenAI Chat Model o AI Agent: selecciona tu credencial de OpenAI.
	4.	Actualizar IDs de la hoja de Google
	•	En los nodos de Google Sheets:
	•	Selecciona el documento correcto (por ID o por lista).
	•	Elige la hoja (por ejemplo Hoja 1) que contiene las columnas de PQRS.
	•	Verifica que los mapeos de columnas (correo, nombre, canal, fecha, tipo, etc.) coincidan con los nombres de tus columnas reales.
	5.	Configurar el Gmail Trigger
	•	En el nodo Gmail PQRS:
	•	Define el buzón o los filtros que usarás (etiqueta, carpeta, etc.).
	•	Haz una prueba de ejecución para asegurarte de que n8n pueda leer correos nuevos.
	6.	Configurar el Webhook (opcional)
	•	En el nodo Webhook:
	•	Copia la URL generada por n8n.
	•	Úsala en Postman o en tu formulario web para enviar solicitudes de prueba.
	7.	Probar el workflow
	•	Envía un correo de prueba al buzón de PQRS o una petición al Webhook.
	•	Ejecuta el workflow (o espera a que el trigger lo dispare automáticamente).
	•	Verifica:
	•	que se cree/actualice la fila correspondiente en Google Sheets, y
	•	que se envíe el correo de respuesta automática.
	8.	Activar en producción
	•	Una vez verificado el funcionamiento, activa el workflow con el botón Activate en n8n.
	•	Opcional: configura variables de entorno en n8n para almacenar API keys y IDs de documentos sin exponerlos en el JSON del workflow.
