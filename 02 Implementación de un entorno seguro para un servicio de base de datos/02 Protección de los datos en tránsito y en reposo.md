# Protección de los datos en tránsito y en reposo

## Introducción

Esta guía resume las principales técnicas de cifrado y control de acceso de red para bases de datos SQL en Azure, tanto en escenarios PaaS (Azure SQL Database, Managed Instance) como IaaS (SQL Server en máquinas virtuales). Incluye recomendaciones sobre cifrado en reposo, gestión de claves con Azure Key Vault y configuración de firewall/NSG.

## Cifrado: en tránsito vs en reposo

- En tránsito: use siempre TLS/SSL para conexiones cliente-servidor (Azure SQL lo requiere por defecto). Habilite versiones modernas de TLS y deshabilite protocolos inseguros.
- En reposo: proteja datos almacenados mediante TDE, cifrado de discos y cifrado de almacenamiento/backups.

## Transparent Data Encryption (TDE)

- Objetivo: cifrar los archivos de base de datos y backups para proteger datos en reposo contra el acceso físico no autorizado.
- PaaS (Azure SQL Database / Managed Instance): TDE viene activado por defecto para bases de datos nuevas. Azure ofrece dos modos de claves:
	- Clave administrada por el servicio (service-managed): es la opción por defecto; Microsoft administra la clave.
	- Clave gestionada por el cliente (Customer-Managed Key, CMK): la clave se almacena en Azure Key Vault (BYOK). Proporciona control sobre la rotación y posibilidad de revocación.
- IaaS (SQL Server en VM): TDE debe habilitarse en la instancia de SQL Server; puede usar certificados locales o integrarse con Key Vault a través de extensiones/gestión de claves.

Integración con Azure Key Vault (TDE CMK):
- Cree un Azure Key Vault y una clave RSA/HSM.
- Conceda permisos al servidor/identidad administrada (Managed Identity) para 'get', 'wrapKey' y 'unwrapKey'.
- Configure el servidor de Azure SQL para usar la clave de Key Vault como `server
tde protector` (portal, CLI o ARM).
- Ventajas: control de ciclo de vida, rotación de claves y registro de acceso. Considere habilitar el registro y la versión de claves.

## Always Encrypted

- Objetivo: proteger columnas sensibles (p. ej., números de tarjeta, SSN) cifrando los valores en el cliente antes de enviarlos a la base de datos. El motor de base de datos nunca ve los valores en claro.
- Componentes:
	- Column Master Key (CMK): almacena y protege las claves de nivel superior (se puede guardar en Azure Key Vault).
	- Column Encryption Key (CEK): usada para cifrar/descifrar los datos de columnas (cifrada por la CMK).
- Tipos de cifrado columnar:
	- Determinístico: permite igualdad y permite índices, pero puede revelar patrones.
	- Aleatorio (randomized): más seguro, no permite búsquedas/equality ni índices.
- PaaS/IaaS: Always Encrypted funciona con Azure SQL Database, Managed Instance y SQL Server en VM siempre que las aplicaciones usen controladores compatibles (Microsoft ADO.NET/ODBC/ODBC drivers con soporte). Azure Key Vault es la opción recomendada para almacenar la CMK en entornos gestionados.
- Consideraciones:
	- Muchas operaciones (búsquedas complejas, LIKE, algunas agregaciones) pueden ser limitadas sobre columnas cifradas.
	- Requiere cambios en la capa de aplicación/driver para que el descifrado ocurra en el cliente.
	- Planifique la rotación de claves y pruebas de compatibilidad antes de desplegar en producción.

## Cifrado en reposo en Azure (visión general)

- Azure cifra datos en reposo en múltiples capas: almacenamiento (SSE), discos (Azure Disk Encryption para VMs), y bases de datos (TDE para Azure SQL y SQL Server backups).
- Modalidades de claves:
	- Platform-managed keys: Azure gestiona las claves para usted.
	- Customer-managed keys (CMK): almacenamiento de claves en Azure Key Vault o en HSMs, mayor control y cumplimiento.
- Backups y snapshots también se cifran; al usar CMK, asegúrese de que la identidad del servicio tenga acceso a Key Vault para desencriptar cuando sea necesario.

## Firewall y control de red

- Azure SQL PaaS (Servidor lógico):
	- Firewall a nivel de servidor: reglas que permiten rangos IP para acceder al servidor.
	- Firewall a nivel de base de datos (virtual network rules): permita subredes de VNet o configure Private Endpoint (Private Link) para acceso privado.
	- Opción "Allow Azure services and resources to access this server": permite que recursos de Azure accedan; evalúe si es necesario.
- Azure SQL Managed Instance: desplegada dentro de una VNet; controle acceso con NSG, Route Tables y reglas de firewall a nivel de red.
- IaaS (SQL Server en VM):
	- Controle el acceso con NSG en la subred/VM y el Firewall de Windows en la máquina virtual.
	- Use JIT (Just-In-Time) para RDP/administración y restringa puertos a orígenes mínimos.

Buenas prácticas de firewall y red:
- Use Private Endpoints/Private Link para eliminar exposición pública donde sea posible.
- Restrinja reglas de entrada a rangos IP mínimos y use identidades gestionadas para servicios que necesiten acceso.
- Habilite la supervisión (NSG flow logs, diagnóstico de firewall de Azure SQL) y alertas por accesos inusuales.

## Recomendaciones y prácticas operativas

- Prefiera CMK en Azure Key Vault para TDE cuando su política de cumplimiento requiera control de claves.
- Use Managed Identity para conceder permisos a Key Vault; evite credenciales embebidas.
- Habilite Always Encrypted para datos sensibles a nivel de columna cuando la aplicación pueda gestionar el descifrado en cliente.
- Planifique la rotación de claves y pruebe la recuperación (restore) con claves CMK y Key Vault configurados.
- Use TLS, Private Link y reglas de firewall estrictas para minimizar la superficie de ataque.
- Registre accesos a Key Vault y auditorías de base de datos para cumplir con requisitos de trazabilidad.

## Recursos útiles

- Documentación TDE y BYOK: https://learn.microsoft.com/azure/azure-sql/
- Always Encrypted: https://learn.microsoft.com/sql/always-encrypted
- Azure Key Vault: https://learn.microsoft.com/azure/key-vault

