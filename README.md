# PIM-JIT-ACCESS-REVIEW
Microsoft Entra ID Governance: Zero Standing Access (ZSA) y Access Review

## Objetivo 
Este proyecto documenta el diseño, implementación y verificación a nivel de auditoría de un marco de **Gobernanza y Administración de Identidades** (**IGA** por sus siglas en inglés). impulsada por principios de **Zero Trust**, enfocándose específicamente en la eliminación de **Privilegios Permanentes (*Standing Privileges*)** en roles administrativos sensibles.
Mediante la integración de **Privileged Identity Management (PIM) for Groups** con **Revisiones de Acceso Automatizadas (*Access Reviews*)**, esta implementación exige accesos temporales Just-In-Time (JIT), flujos de aprobación, autenticación multifactor (MFA) y un mecanismo de recertificación de accesos seguro ante fallos para prevenir la acumulación de permisos (*Privilege Creep*).

---

## 🏗️ Arquitectura y Ciclo de Vida de Gobernanza

---

### 00-Prerrequisitos y Configuración de Objetos Base ###

* **Creación de Grupo Habilitado para Roles (*Role-assignable group*):**
   - Se creó el grupo de seguridad `User Admins` y habilitamos el campo ”Microsoft Entra roles can be assigned to the group”
   - 
   * *Nota de Arquitectura:* Los grupos habilitados para roles previenen que administradores de grupos no autorizados modifiquen la membresía, añadiendo una capa extra de protección.

* **Asignación de Rol de Entra ID al Grupo:**
   - Se asignó el rol **User Administrator** 
- - - 

### 01-Acceso Just-In-Time (JIT) vía PIM para Grupos 

### Configuracion PIM 

Dentro del apartado “Privileged Identity Management” ingresamos a “Groups” 
En el apartado “Settings” ingresamos  a “member” 	
ingresamos “Edit” para configurar los parámetros de seguridad de PIM 

En el apartado “Activation” establecemos las siguientes configuraciones para cuando el usuario eleva/activa el rol:
La activación durará un máximo de 4 horas 
Se necesita MFA para poder activarse
Se necesita una justificación en la solicitud de activacion
Se necesita la aprobación de un Usuario 

Vamos a seleccionar a “Josué Rivero” como el user encargado de aprobar las solicitudes. 
-

En el apartado “Assignment” vamos a establecer las reglas para que el usuario sea elegible o para la activación de este cuando se lo asigna al grupo. 
“Allow permanent eligible assignment” **Deshabilitado** para evitar que un user sea elegible de forma permanente/indefinida
“Expire eligible assignments after” establecemos 1 año. esto significa que pasado este año la elegibilidad para solicitar el rol vence  
“Allow permanent active assignment”  **Deshabilitado** No se puede asignar a un usuario como miembro activo permanente del grupo sin pasar por PIM 
“Expire active assignments after” Establecemos 6 meses. esto significa que si un admin hace una asignación directa (sin hacerla vía JIT) el usuario activo solo va a tener acceso durante 6 meses.

### Agregar Member Elegible

Dentro del apartado “Privileged Identity Management” ingresamos a “Groups” 
Dentro del grupo vamos al apartado “assignments” e ingresamos a “Add Assignments”
En “Select Roles” vamos a elegir la opción “member” y seleccionamos al user “PruebaUserAdmin”
Dentro del Apartado “Setting” establecemos que el user sera del tipo “Elegible” durante 1 año




### 02-Access Recertification & Automated Remediation 
Ingresamos a Access Review y seleccionamos “New Access Review”
Seleccionamos el template “Resource Review”
En el apartado “Review Type” seleccionamos que está esta destinada esta revisión, puede estar dirigida a aplicaciones o a grupos.
“Select what to review” seleccionamos “Teams + Groups”
“Review Scopes” seleccionamos “Select Teams + Groups” y agregamos el grupo “User Admins”

Scope: Seleccionamos “All users”

En el apartado “Reviews” vamos a establecer quién es el encargado de la toma de decisiones de recertificación y configuramos la duración y recurrencia de la revisión 
Seleccionamos a “Josué Rivero” como el Reviewer
“Duration”: a fin de demostrar cómo funciona el Access Review en este proyecto, nos limitamos a 1 día. En este campo se establece cuantos días tiene el reviewer para dar una respuesta.
“Review Recurrence” a fin de demostrar cómo funciona el Access Review en este proyecto, nos limitamos a 1 vez. En este campo se establece cada cuanto tiempo comienza una campaña de revisión 
 “Start Date” 10/26/2026 establece el dia en el que comienza la campaña
En el apartado “Settings” vamos a establecer el comportamiento automatizado y las reglas de remediación cuando finaliza la campaña de revisión. 
“Auto apply results to rescue” Habilitado, establecemos que los cambios se apliquen automaticamente una vez termine la campaña
“if reviewers don’t respond” Seleccionamos “Remove Access” en este apartado establecemos que  si el reviewer no da respuesta automaticamente se eliminen los accesos.
Establecemos que a Josué Rivero le llegue una notificacion cuando termine la revision 
establecemos “ayudas” para que el reviewer pueda tomar decisiones. “No sign -in within 30 days” va a sugerir denegar o eliminar accesos a user que no hayan iniciado sesion en 30 dias
“Advanced Settings" establecemos recordatorios, notificaciones, contexto para el reviewer y habilitamos “Justification required” para obligar al review a dar un justificativo sobre sus decisiones. 

Review + Create es la etapa final, aca estabelecemos un nombre, la descripcion de la campaña de revision y reconfirmamos nuestras configuraciones
“Review Name” será “UserAdmins-Review”	



## Verificación

### Escenario 1 - JIT Acceso Concedido  
En este primer escenario vamos a simular un pedido de acceso al grupo UserAdmins via JIT. Vamos a solicitar acceso durante 1 hora

**Solicitud**

Con el user PruebaUserAdmin vamos a ingresar a “Privileged Identity Management” y buscamos el grupo “User Admins”
dentro del grupo en “My roles” en el apartado “Eligible Assignments” encontramos el rol a solicitar seleccionando “Activate”
Seleccionamos 1 hora y agregamos el justificativo para la solicitud 
por ultimo, tenemos la notificación de que enviamos la solicitud correctamente

**Acceso Concedido**

Con el user de Josue Rivero vamos a ingresar a “Privileged Identity Management” y seleccionamos “Approve requests”
una vez dentro, ingresamos a Groups veremos la solicitud de PruebaUserAdmin. vamos a concederle el acceso con el boton “Approve”
Ponemos el justificativo de la aprobacion y finalizamos con el boton “Confirm”
*Verificacion de acceso concedido*
Desde PruebaUserAdmin en “Privileged Identity Management” vamos a “my role” y en “Groups” vemos que ya se encuentra en “Active Assignments” podemos ver la hora de vencimiento del acceso.
Desde Josué Rivero inmediatamente después de conceder el acceso vemos al user en el apartado “Active assignments” y no en “Elegible  assignments” 

--- 

### Escenario 2: JIT Vencido 
A fin de confirmar que el JIT este funcionando correctamente. una vez vencido la hora de acceso, pasamos a confirmar que el user “PruebaUserAdmin” ya no tenga acceso al grupo “UserAdmins”

* **Horade Activacion:** `10/26/2026, 6:51:25 PM` 
* **Hora de Vencimiento:** `10/26/2026, 7:51:25 PM` 
* **El Log muestra la siguiente informacion:** `Remove member from role (PIM activation expired)` 

--- 

### 🔹 Phase 3: Access Review

Para este proyecto vamos a tomar 2 escenarios con el Access Review
Escenario 1: El reviewer “Josue Rivero” Concede el acceso en tiempo y forma
Escenario 2: El reviewer “Josue Rivero” no responde a la revisión

**Escenario 1: El reviewer “Josue Rivero” Concede el acceso en tiempo y forma**

Vemos el estado de la campaña, en la misma vemos que todavia no hay respuesta sobre 1 usuario. 
Dentro del perfil “My Access” del user “Josue Rivero” tiene como Pendiente dar respuesta a la revision
Vemos el user “PruebaUserAdmin”, la recomendación (Aprobar), el boton de “details” y el botones de acción (Approve, Deny etc.). 
Vamos a aprobar y añadir el justificactivo
Automaticamente vemos los cambios
Tambien se actualizo la informacion en Microsoft Entra ID

**Escenario 2: El reviewer “Josue Rivero” no responde a la revisión**

Ante la falta de respuesta, vemos que el estado del access review cambio. paso de tener usuarios en “Not Reviewed” (Sin respuesta) a pasarlos a “Deny” (acceso denegado)
en los Logs veremos como cuando no detecto respuesta ejecuto la regla que configuramos de denegar el acceso.


* **Audit Log Events:** 
Configuracion y creacion de PIM 

Momento en el cual PIM es correctamente habilitado para el grupo UserAdmins

inicia el proceso para conceder la condición de Elegible usuario PruebaUserAdmin

Confirmacion de que el user PruebaUserAdmin puede ser Elegible para integrar el grupo UserAdmins de forma temporal via PIM y JIT


En este log se documente las modificaciones en el rol member dentro del grupo User Admins (confguraciones tales como “Duracion Maxima”,”Activacion con MFA” y aprobación via user “Josue Rivero”)




**Logs de Ingreso Vía JIT**

Los registros de auditoría de PIM for Groups se emiten secuencialmente según la finalización de transacciones en el backend de Microsoft Entra:

Acceso Concedido

Vemos la aprobación del user Josué Rivero 


Tenemos el ingreso exitoso del user PruebaUserAdmin al grupo UserAdmins vía JIT. ademas de detallar la justificación de la solicitud de ingreso



Acceso Vencido 

Una vez cumplida la hora de acceso vía JIT, se elimino al usuario del mismo de forma automatizada
	


Access Review 

Vemos los logs del Access Review, desde el overview vemos la creacion


Vemos el log del Escenario 1 en cual el reviewer aprueba el usuario



Vemos los logs del escenario 2. en el cual el reviewer no da respuesta a la revision 
Vemos como termino el tiempo de la revision 


Vemos como al no tener respuesta del reviewer se activa el Auto-Review para evaluar las configuraciones


—

 ## 💡 Key Takeaways & Lessons Learned 

1. **Zero Standing Access is Attainable:** Cloud identities do not need permanent administrative rights. Enforcing JIT access drastically shrinks the attack surface (*Blast Radius*). 
2. **Governance Must Be Fail-Safe:** Access Reviews should always default to access removal upon reviewer inactivity, ensuring security takes precedence over convenience. 3. **Auditability is Essential:** PIM and Access Reviews generate clear, immutable audit trails vital for SOC monitoring and regulatory compliance audits. 
