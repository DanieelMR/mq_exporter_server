# IBM MQ Exporter TLS Configuration

Entorno Docker para probar y configurar IBM MQ Exporter con conexiones TLS/SSL.

## Descripción

Este proyecto proporciona:
- Entorno Docker con IBM MQ Server y Client para pruebas TLS
- Configuración de CCDT (Client Channel Definition Table) para conexiones seguras
- Plantillas de servicio systemd para mq_exporter

## Configuración

### 1. Crear archivo CCDT

Copiar el archivo de ejemplo y configurar con tus datos:

```bash
cp ccdt.example.json ccdt.json
```

Editar `ccdt.json` con los valores de tu entorno:
- `host`: Hostname o IP del servidor MQ
- `port`: Puerto del listener MQ
- `queueManager`: Nombre del Queue Manager
- `name`: Nombre del canal SVRCONN
- `certificateLabel`: Label del certificado en el keystore

### 2. Crear servicio systemd

Copiar el archivo de ejemplo:

```bash
sudo cp mq_exporter.service.example /etc/systemd/system/mq_exporter.service
```

Editar con tus valores:
- `MQSSLKEYR`: Ruta al keystore (sin extensión .kdb)
- `-ibmmq.ccdtUrl`: Ruta al archivo ccdt.json
- `-ibmmq.queueManager`: Nombre del Queue Manager
- `-ibmmq.monitoredQueues`: Patrón de colas a monitorear

### 3. Activar servicio

```bash
sudo systemctl daemon-reload
sudo systemctl enable mq_exporter.service
sudo systemctl start mq_exporter.service
sudo systemctl status mq_exporter.service
```

## Parámetros importantes del mq_exporter

| Parámetro | Descripción |
|-----------|-------------|
| `-ibmmq.ccdtUrl` | Ruta al archivo CCDT |
| `-ibmmq.queueManager` | Nombre del Queue Manager |
| `-ibmmq.monitoredQueues` | Patrón de colas (ej: `'MY_QUEUE.*'`) |
| `-ibmmq.monitoredChannels` | Patrón de canales (ej: `'*'`) |
| `-ibmmq.useStatus` | Habilita métricas de status |
| `-ibmmq.usePublications` | Usar publicaciones (false para plataformas antiguas) |
| `-ibmmq.showInactiveChannels` | Mostrar canales inactivos |

## Variables de entorno

| Variable | Descripción |
|----------|-------------|
| `MQSSLKEYR` | Ruta al keystore SSL (sin extensión) |
| `MQCERTLABL` | Label del certificado (opcional si está en CCDT) |

## Métricas de canales

Para obtener métricas `ibmmq_channel_*`:

```
-ibmmq.monitoredChannels='*' -ibmmq.useStatus='true'
```

Estados de canal:
- `RUNNING` (3)
- `STOPPED` (4)
- `RETRYING` (6)
- `INACTIVE` (0) - requiere `-ibmmq.showInactiveChannels`
- `BINDING` (5) - requiere `-ibmmq.showInactiveChannels`

## Docker - Entorno de pruebas

```bash
# Iniciar
docker-compose up -d

# Probar conexión TLS
docker exec mq-client bash -c "cd /opt/mqm/samp/bin && ./amqssslc -m YOUR_QMGR -c YOUR_CHANNEL -x 'hostname(1414)' -s TLS_RSA_WITH_AES_256_CBC_SHA256 -k /certs/key -l your-cert-label"

# Detener
docker-compose down
```

## Archivos

| Archivo | Descripción |
|---------|-------------|
| `ccdt.example.json` | Plantilla CCDT |
| `mq_exporter.service.example` | Plantilla servicio systemd |
| `docker-compose.yml` | Entorno Docker de pruebas |
