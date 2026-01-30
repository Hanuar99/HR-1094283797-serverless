# Solución Serverless – Registro de Usuarios y Auditoría

## Descripción
Esta solución implementa una arquitectura serverless en AWS que expone una API
para la creación de usuarios y genera eventos para registrar auditoría de manera
asíncrona, cumpliendo con los requisitos de la prueba técnica.

---

## Arquitectura
La solución está compuesta por los siguientes servicios:

- Amazon API Gateway
- AWS Lambda (Python)
- Amazon DynamoDB
- Amazon Kinesis Data Streams
- Amazon CloudWatch Logs

El flujo completo se encuentra documentado en el diagrama de arquitectura
incluido en este repositorio.

---

## Recursos AWS creados

Todos los recursos están identificados con las iniciales y documento del autor:

### API Gateway
- `POST /usuarios`

### Lambda Functions
- `HR_1094283797_usuarios_lambda`
- `HR_1094283797_auditoria_lambda`

### DynamoDB Tables
- `HR_1094283797_usuarios_tabla`
- `HR_1094283797_auditoria_usuario`

### Kinesis Data Stream
- `HR_1094283797_usuarios_stream`

### Logging
- Amazon CloudWatch Logs habilitado para ambas Lambdas

---

## PARTE 1: Registro de Usuarios

El endpoint `POST /usuarios` recibe un payload estricto con los campos:

- nombre
- apellido
- documento
- telefono

### Reglas implementadas
- Validación estricta del payload (HTTP 400 si faltan o sobran campos).
- El campo `documento` es único.
- Manejo de error controlado si el usuario ya existe.
- Registro del usuario en DynamoDB con:
  - documento (Partition Key)
  - nombre
  - apellido
  - telefono
  - fecha_registro (epoch en milisegundos)

Al crear un usuario exitosamente, se emite un evento `USUARIO_CREADO` en Kinesis
Data Streams.

---

## PARTE 2: Evento y Auditoría

Los eventos del stream `usuarios-stream` activan una segunda Lambda que:

- Procesa únicamente eventos con tipo `USUARIO_CREADO`.
- Utiliza `documento` como partition key.
- Registra la auditoría en DynamoDB con los campos:
  - documento
  - fecha_registro
  - processed_at

---

## Logging y Observabilidad

Ambas Lambdas registran logs en Amazon CloudWatch Logs por cada ejecución,
incluyendo como mínimo los siguientes campos:

- `code`: "OK" si la ejecución fue exitosa, "NOK" si ocurrió un error.
- `error_message`: mensaje de error comprensible.
- `error_technical`: detalle técnico de la excepción capturada.

---

## Pruebas realizadas
- Creación de usuario válida.
- Payload inválido (HTTP 400).
- Documento duplicado (error controlado).
- Verificación de registros en DynamoDB Usuarios.
- Verificación de registros en DynamoDB Auditoría.
