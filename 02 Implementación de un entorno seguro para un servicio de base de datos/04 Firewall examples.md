# Ejemplos de Firewall mediante comandos (T-SQL)

En Azure SQL puede gestionar reglas de firewall a nivel de base de datos y a nivel de servidor. Las reglas a nivel de servidor (server-level firewall rules) también pueden verse y modificarse desde el portal de Azure en la configuración del servidor SQL.

A continuación se muestran consultas y procedimientos almacenados T-SQL útiles para listar, crear y eliminar reglas. Comentarios en castellano explican cada comando; sustituya las direcciones IP de ejemplo por las reales.

```sql
-- Mostrar las reglas de firewall específicas de la base de datos actual
-- Devuelve las reglas definidas con sp_set_database_firewall_rule
SELECT * FROM sys.database_firewall_rules
GO

-- Crear o actualizar una regla de firewall a nivel de base de datos
-- @name: nombre de la regla; @start_ip_address y @end_ip_address: rango permitido
EXECUTE sp_set_database_firewall_rule 
	@name = N'AWFirewallRule',
	@start_ip_address = '000.000.000.000', 
	@end_ip_address = '000.000.000.000'
GO

-- Eliminar una regla de firewall a nivel de base de datos por su nombre
EXECUTE sp_delete_database_firewall_rule
	@name = N'AWFirewallRule'
GO

-- Mostrar las reglas de firewall a nivel de servidor
-- Estas afectan al servidor lógico y abarcan todas las bases de datos del servidor
SELECT * FROM sys.firewall_rules
GO

-- Crear o actualizar una regla de firewall a nivel de servidor
-- @name: nombre de la regla; @start_ip_address y @end_ip_address: rango permitido
EXECUTE sp_set_firewall_rule 
	@name = N'AWFirewallRule',
	@start_ip_address = '000.000.000.000', 
	@end_ip_address = '000.000.000.000'
GO

-- Eliminar una regla de firewall a nivel de servidor por su nombre
EXECUTE sp_delete_firewall_rule
	@name = N'AWFirewallRule'
GO
```

Notas y buenas prácticas:

- Reemplace `000.000.000.000` por la IP o rango real que quiera autorizar.
- Prefiera reglas de rango lo más restrictivas posible; evite abrir acceso público amplio.
- Para entornos Azure, considere usar Private Endpoint/Private Link en lugar de abrir IPs públicas.
- Las reglas de servidor pueden gestionarse desde el portal de Azure (Azure SQL -> Configuración -> Firewall y redes virtuales), CLI o ARM templates.
- Supervise cambios y registre auditoría para detectar reglas añadidas o modificadas de forma inesperada.

