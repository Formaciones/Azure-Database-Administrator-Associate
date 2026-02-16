# Implementación de un entorno seguro para un servicio de base de datos

Este indice resume los contenidos disponibles en esta carpeta.


## Contenidos

- [Autenticacion en Azure SQL: IaaS vs PaaS](01%20Configure%20database%20authentication%C2%A0and%20authorization.md)
  - Explica los metodos de autenticacion para SQL Server en IaaS y Azure SQL Database en PaaS, con recomendaciones de seguridad y autorizacion.

- [Ejemplos de autenticacion y autorizacion (SQL)](02%20Authorization%20examples.md)
  - Reune ejemplos ordenados de creacion de usuarios, asignacion de permisos, roles, esquemas y pruebas de acceso.

- [TDE, Always Encrypted y Firewall (PaaS e IaaS)](03%20TDE%2C%20Always%20Encrypted%2C%20y%20Firewall%20Server%20y%20Database.md)
  - Explica conceptos y diferencias entre TDE y Always Encrypted, integración con Key Vault y ejemplos de firewall para Azure SQL en escenarios PaaS e IaaS.

- [Ejemplos de Firewall (T-SQL)](04%20Firewall%20examples.md)
  - Contiene comandos T-SQL para listar, crear y eliminar reglas de firewall a nivel de base de datos y servidor, con comentarios en castellano.

- [TDE en Azure SQL Database (PaaS)](05%20TDE%20en%20Azure%20SQL%20Database%20PaaS.md)
  - Comandos y buenas prácticas para gestionar TDE en Azure SQL Database (PaaS) y uso de claves gestionadas por el cliente (CMK) en Key Vault.

- [TDE en SQL Server IaaS (VM)](06%20TDE%20en%20SQL%20Server%20IaaS.md)
  - Procedimientos T-SQL para crear certificados, DEK, activar TDE, y pasos de respaldo y recuperación del certificado en entornos IaaS.
