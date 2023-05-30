# Diagrama de dependencias KrezkaMas

Este documento presenta un diagrama de las dependencias de diferentes proyectos e infraestructura de Krezka.

## Login

El siguiente diagrama describe el proceso de Login

```mermaid
sequenceDiagram
    participant FrontEnd
    participant BackEnd
    participant DBKrezca
    participant APISeguridad
    participant BDSeguridad
    participant Redis
    participant CFDIGlobal(Masteredi)
    participant Queue(SA)
    participant AzureFunction
    participant TableStorage(SA)
    participant BlobStorage(SA)
    participant CP(Masteredi)

    FrontEnd->>Backend: login(usuario,contraseña)
    Backend->>APISeguridad: validaUsuario()
    APISeguridad->>DBSeguridad: getData()
    DBSeguridad->>APISeguridad: Data
    APISeguridad->>Backend: success()
    Backend->>Redis: returnDetail()
```

## Generar Cliente

El siguiente diagrama describe el proceso de crear un cliente

```mermaid
sequenceDiagram
    participant FrontEnd
    participant BackEnd
    participant DBKrezca
    participant APISeguridad
    participant BDSeguridad
    participant Redis
    participant CFDIGlobal(Masteredi)
    participant Queue(SA)
    participant AzureFunction
    participant TableStorage(SA)
    participant BlobStorage(SA)
    participant CP(Masteredi)

    FrontEnd->>Backend: insertarCliente()
    Backend->>CP(Masteredi): obtenerDirecciones(CP)
    CP(Masteredi)->>Backend: Direccion
    Backend->>DBKrezka: Cliente
    BDKrezka->>Backend: success()

```

## Generar Factura

El siguiente diagrama describe el proceso de creación de una factura.

```mermaid
sequenceDiagram
    participant FrontEnd
    participant BackEnd
    participant DBKrezca
    participant APISeguridad
    participant BDSeguridad
    participant Redis
    participant CFDIGlobal(Masteredi)
    participant Queue(SA)
    participant AzureFunction
    participant TableStorage(SA)
    participant BlobStorage(SA)
    participant CP(Masteredi)

    FrontEnd->>Backend: generarFactura()
    Backend->>Redis: validarPermisos()
    Redis->>Backend: Permisos
    loop Factura Intermedia
        Backend-->Backend: generarIntermedio()
    end
    Backend->>CFDIGlobal: generaCFDI()
    CFDIGlobal-->Backend: CFDI-xml
    loop Sello
        Backend-->Backend: sellarCFDI()
    end
    Backend->>CFDIGlobal: certificar()
    CFDIGlobal->>Backend: xmlCertificado
    loop PDF
        Backend-->Backend: generaPDF()
    end
    Backend->>Queue(SA): enviaCorreo()
    Queue(SA)->>AzureFuntion: trigger()
    Backend->>DBKrezka: registraOperacion()
    AzureFunction->>DBKrezka: Procesa Póliza General(trigger cada 5min)
    DBKrezka->>AzureFunction: Procesa Póliza General: actualizaTabla
    AzureFuncion->>TableStorage(SA): registroNuevo
    Backend->>FrontEnd: success()
```
