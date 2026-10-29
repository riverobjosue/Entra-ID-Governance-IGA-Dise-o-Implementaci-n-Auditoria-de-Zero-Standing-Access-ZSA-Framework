# PIM-JIT-ACCESS-REVIEW
Microsoft Entra ID Governance: Zero Standing Access (ZSA) y Access Review

## Objetivo 
Este proyecto documenta el diseño, implementación y verificación a nivel de auditoría de un marco de **Gobernanza y Administración de Identidades** (**IGA** por sus siglas en inglés). impulsada por principios de **Zero Trust**, enfocándose específicamente en la eliminación de **Privilegios Permanentes (*Standing Privileges*)** en roles administrativos sensibles.
Mediante la integración de **Privileged Identity Management (PIM) for Groups** con **Revisiones de Acceso Automatizadas (*Access Reviews*)**, esta implementación exige accesos temporales Just-In-Time (JIT), flujos de aprobación, autenticación multifactor (MFA) y un mecanismo de recertificación de accesos seguro ante fallos para prevenir la acumulación de permisos (*Privilege Creep*).

---

 Arquitectura y Ciclo de Vida de Gobernanza

---

### 00-Prerrequisitos y Configuración de Objetos Base ###

* **Creación de Grupo Habilitado para Roles (*Role-assignable group*):**
   - Se creó el grupo de seguridad `User Admins` y habilitamos el campo ”Microsoft Entra roles can be assigned to the group”
  
     <img width="747" height="831" alt="00-CreacionGrupo-Owner" src="https://github.com/user-attachments/assets/2591b775-df45-4a18-a4ac-f614dd2ff778" />

* **Asignación de Rol de Entra ID al Grupo:**
   - Se asignó el rol **User Administrator** 
<img width="851" height="879" alt="00-CreacionGrupo-Rol" src="https://github.com/user-attachments/assets/c47050b0-fefb-4d71-a77d-156c39adc100" />
<img width="737" height="848" alt="00-CreacionGrupo" src="https://github.com/user-attachments/assets/7ab2b215-646b-4089-b86e-0e27224a3854" />


- - - 

### 01-Acceso Just-In-Time (JIT) vía PIM para Grupos 

### Configuracion PIM 

Dentro del apartado “Privileged Identity Management” ingresamos a “Groups” 

<img width="1237" height="396" alt="01-PIM-Group" src="https://github.com/user-attachments/assets/f7a8ac9d-4cb3-4d5a-a7b8-d5aeb090f48e" />

En el apartado “Settings” ingresamos  a “member” 	

<img width="1572" height="355" alt="01-PIM-Member" src="https://github.com/user-attachments/assets/60d3bcaf-b319-4124-accc-d13b712e4a02" />


Ingresamos “Edit” para configurar los parámetros de seguridad de PIM 

<img width="1435" height="852" alt="01-PIM-Member-Conf" src="https://github.com/user-attachments/assets/63d6f622-39f7-4a9f-87fc-9b736fd91385" />

En el apartado “Activation” establecemos las siguientes configuraciones para cuando el usuario eleva/activa el rol:
- La activación durará un máximo de 4 horas 
- Se necesita MFA para poder activarse
- Se necesita una justificación en la solicitud de activacion
- Se necesita la aprobación de un Usuario 

<img width="553" height="856" alt="01-PIM-CONF-ACTIV" src="https://github.com/user-attachments/assets/24f75513-9f18-41bd-acfb-ab5fc05f4994" />

Vamos a seleccionar a “Josué Rivero” como el user encargado de aprobar las solicitudes. 

<img width="1101" height="864" alt="01-PIM-Overviewer" src="https://github.com/user-attachments/assets/02dfd97d-23a7-4486-a2c2-53fb1f58e246" />

En el apartado “Assignment” vamos a establecer las reglas para que el usuario sea elegible o para la activación de este cuando se lo asigna al grupo. 
“Allow permanent eligible assignment” **Deshabilitado** para evitar que un user sea elegible de forma permanente/indefinida
“Expire eligible assignments after” establecemos 1 año. esto significa que pasado este año la elegibilidad para solicitar el rol vence  
“Allow permanent active assignment”  **Deshabilitado** No se puede asignar a un usuario como miembro activo permanente del grupo sin pasar por PIM 
“Expire active assignments after” Establecemos 6 meses. esto significa que si un admin hace una asignación directa (sin hacerla vía JIT) el usuario activo solo va a tener acceso durante 6 meses.

<img width="524" height="849" alt="01-PIM-CONF-ASSIGNMENT" src="https://github.com/user-attachments/assets/6a92a06a-9a29-4815-8820-c1a7099fb34b" />

### Agregar Member Elegible

Dentro del apartado “Privileged Identity Management” ingresamos a “Groups” 

<img width="1237" height="396" alt="01-PIM-Group" src="https://github.com/user-attachments/assets/f7a8ac9d-4cb3-4d5a-a7b8-d5aeb090f48e" />

Dentro del grupo vamos al apartado “assignments” e ingresamos a “Add Assignments”

<img width="1574" height="321" alt="01-PIM-ADDASSIGNMENTS" src="https://github.com/user-attachments/assets/72cd36e1-f923-4e4c-8188-a217ebad9683" />

En “Select Roles” vamos a elegir la opción “member” y seleccionamos al user “PruebaUserAdmin”

<img width="552" height="462" alt="01-PIM-ASSIGNMENT-MEMBER" src="https://github.com/user-attachments/assets/8135088f-ee59-449f-aa1b-02b801becaf4" />
<img width="520" height="845" alt="01-PIM-ASSIGNMENT-MEMBERSHIP" src="https://github.com/user-attachments/assets/ff207952-79e7-4822-a1a6-56e19797b958" />

Dentro del Apartado “Setting” establecemos que el user sera del tipo “Elegible” durante 1 año

<img width="519" height="843" alt="01-PIM-ASSIGNMENT-SETTING" src="https://github.com/user-attachments/assets/69451987-bee7-40a3-baa5-d5eaec551c2b" />

### 02-Access Recertification & Automated Remediation 
Ingresamos a Access Review y seleccionamos “New Access Review”

<img width="1588" height="681" alt="02-AccessReview-New" src="https://github.com/user-attachments/assets/e3026eef-d669-47c3-98e4-fb2cfefd3e6a" />

Seleccionamos el template “Resource Review”

<img width="748" height="448" alt="02-AccessReview-ResourceReview" src="https://github.com/user-attachments/assets/d43460ef-d629-4f80-ba42-5ebd9157cab4" />

En el apartado “Review Type” seleccionamos para qué quién esta destinada esta revisión, puede estar dirigida a aplicaciones o a grupos.
“Select what to review” seleccionamos “Teams + Groups”

<img width="778" height="847" alt="02-AccessReview-ReviewType(1)" src="https://github.com/user-attachments/assets/159f69ac-686f-4f73-b74b-686bfbea2d63" />

“Review Scopes” seleccionamos “Select Teams + Groups” y agregamos el grupo “User Admins”
<img width="1098" height="871" alt="02-AccessReview-ReviewType-Group" src="https://github.com/user-attachments/assets/fdbdd164-0ece-4ff7-bbd0-35a842662a85" />

Scope: Seleccionamos “All users”

<img width="823" height="839" alt="02-AccessReview-ReviewType" src="https://github.com/user-attachments/assets/629718c2-3941-444e-a636-7b485324368b" />

En el apartado “Reviews” vamos a establecer quién es el encargado de la toma de decisiones de recertificación, también configuramos la duración y recurrencia de la revisión. 

- Seleccionamos a “Josué Rivero” como el Reviewer
- “Duration”: a fin de demostrar cómo funciona el Access Review en este proyecto, nos limitamos a 1 día. En este campo se establece cuantos días tiene el reviewer para dar una respuesta.
- “Review Recurrence” a fin de demostrar cómo funciona el Access Review en este proyecto, nos limitamos a 1 vez. En este campo se establece cada cuanto tiempo comienza una campaña de revisión 
- “Start Date” 10/26/2026 establece el dia en el que comienza la campaña

 <img width="591" height="578" alt="02-AccessReview-Reviews" src="https://github.com/user-attachments/assets/4256d701-5bb4-49bd-b1bc-652df4cd7d4f" />

En el apartado “Settings” vamos a establecer el comportamiento automatizado y las reglas de remediación cuando finaliza la campaña de revisión.

- “Auto apply results to rescue” Habilitado, establecemos que los cambios se apliquen automaticamente una vez termine la campaña
- “if reviewers don’t respond” Seleccionamos “Remove Access” en este apartado establecemos que  si el reviewer no da respuesta automaticamente se eliminen los accesos.
- Establecemos que a Josué Rivero le llegue una notificación cuando termine la revisión 
- Establecemos “ayudas” para que el reviewer pueda tomar decisiones. “No sign -in within 30 days” va a sugerir denegar o eliminar accesos a user que no hayan iniciado sesion en 30 dias
- “Advanced Settings" establecemos recordatorios, notificaciones, contexto para el reviewer y habilitamos “Justification required” para obligar al review a dar un justificativo sobre sus decisiones. 

<img width="621" height="791" alt="02-AccessReview-Settings" src="https://github.com/user-attachments/assets/7ffaa779-b28a-4e8a-9f17-5a6db738d2b5" />

Review + Create es la etapa final, aca estabelecemos el nombre y la descripción de la campaña de revisión, también reconfirmamos nuestras configuraciones

<img width="525" height="844" alt="02-AccessReview-Review+Create" src="https://github.com/user-attachments/assets/b64f0d76-800d-45b6-8d0e-502dbbf791a3" />

## Verificación

### Escenario 1 - JIT Acceso Concedido  
En este primer escenario vamos a simular un pedido de acceso al grupo UserAdmins via JIT durante 1 hora. 

**Solicitud**

Con el user PruebaUserAdmin vamos a ingresar a “Privileged Identity Management” y buscamos el grupo “User Admins”

<img width="1572" height="388" alt="01-JIT-Roles-GRPUserAdmins" src="https://github.com/user-attachments/assets/41631180-f108-45b5-9e23-dc5b94197a33" />

dentro del grupo en “My roles” en el apartado “Eligible Assignments” encontramos el rol a solicitar seleccionando “Activate”

<img width="1584" height="261" alt="01-JIT-UserAdmins" src="https://github.com/user-attachments/assets/1a58ba9a-cde7-40a6-858f-a21aa0d70474" />

Seleccionamos 1 hora y agregamos el justificativo para la solicitud 

<img width="578" height="880" alt="01-JIT-Solicitud" src="https://github.com/user-attachments/assets/008a6b17-46ad-4c07-a35a-13d09c74b867" />

**Acceso Concedido**

Con el user de Josue Rivero vamos a ingresar a “Privileged Identity Management” y seleccionamos “Approve requests”

<img width="1572" height="513" alt="01-JIT-OVERVIEW" src="https://github.com/user-attachments/assets/a3a890f8-66f7-4ffa-b850-cbed23a8fc33" />

una vez dentro, ingresamos a Groups veremos la solicitud de PruebaUserAdmin. vamos a concederle el acceso con el boton “Approve”

<img width="1319" height="218" alt="01-JIT-ApproveOverview" src="https://github.com/user-attachments/assets/1e9f01c5-d47b-4fa2-ba2b-91d6b9aa7499" />

Ponemos el justificativo de la aprobación y finalizamos con el boton “Confirm”

<img width="343" height="873" alt="01-JIT-ApproveJustificacion" src="https://github.com/user-attachments/assets/e09b88e4-3bd2-428d-a140-8bfea6a48285" />

**Verificacion de acceso concedido**

Desde el user "PruebaUserAdmin" en “Privileged Identity Management” vamos a “my role” y en “Groups” confirmamos que el rol ya se encuentra en “Active Assignments”. Podemos ver la hora de vencimiento del acceso.

<img width="1587" height="489" alt="01-JIT-PruebaUserAdmin-Overview" src="https://github.com/user-attachments/assets/88d685cb-64b0-44d9-9f5e-5949e782f409" />
<img width="1572" height="245" alt="01-JIT-PruebaUserAdmin-Active" src="https://github.com/user-attachments/assets/a83c1905-339b-4743-b600-fa4ff1bb35ca" />

Desde el user "Josué Rivero" inmediatamente después de conceder el acceso confirmamos que el user se visualiza en el apartado “Active assignments” y no en “Elegible  assignments” 

<img width="1592" height="465" alt="01-JIT-JosueRivero-Overview" src="https://github.com/user-attachments/assets/c8014d04-0395-4d75-aa1c-b6771d3cd9a3" />


--- 

### Escenario 2: JIT Vencido 
A fin de confirmar que el JIT este funcionando correctamente. una vez vencido el plazo de acceso, pasamos a confirmar que el user “PruebaUserAdmin” ya no tenga acceso al grupo “UserAdmins”

* **Hora de Activacion:** `10/26/2026, 6:51:25 PM` 
* **Hora de Vencimiento:** `10/26/2026, 7:51:25 PM` 
* **El Log muestra la siguiente informacion:** `Remove member from role (PIM activation expired)` 

--- 

### 02- Access Review

Para este proyecto vamos a tomar 2 escenarios con el Access Review
Escenario 1: El reviewer “Josue Rivero” Concede el acceso en tiempo y forma
Escenario 2: El reviewer “Josue Rivero” no responde a la revisión

**Escenario 1: El reviewer “Josue Rivero” Concede el acceso en tiempo y forma**

Vemos el estado de la campaña, en la misma vemos que todavia no hay respuesta sobre 1 usuario. 

<img width="1583" height="545" alt="02-AccessReview-Overview-EntraIDpng" src="https://github.com/user-attachments/assets/cadb57af-6881-46d6-9c49-d87423f94bef" />

Dentro del perfil “My Access” del user “Josue Rivero” tiene como Pendiente dar respuesta a la revisión

<img width="916" height="372" alt="02-AccessReview-MyAccess-Overview" src="https://github.com/user-attachments/assets/67d8aaa2-15ad-4c93-b91b-1c8354084291" />

Vemos el user “PruebaUserAdmin”, la recomendación (Aprobar), el boton de “details” y el botones de acción (Approve, Deny etc.). 

<img width="1549" height="327" alt="02-AccessReview-Approve" src="https://github.com/user-attachments/assets/d21b06fc-cf82-4d9e-8981-a535343bd8a0" />

Vamos a aprobar y añadir el justificactivo

<img width="430" height="452" alt="02-AccessReview-Approve-Justificacion" src="https://github.com/user-attachments/assets/6ffe40e6-3a0b-45f6-acae-9d1c9f5fc429" />

Automáticamente vemos los cambios

<img width="1570" height="334" alt="02-AccessReview-Approve Overview" src="https://github.com/user-attachments/assets/df6666f5-02ea-4982-b7ad-8551e7e53664" />

Tambien se actualizo la informacion en Microsoft Entra ID

<img width="1568" height="577" alt="02-AccessReview-Approve-Overview-EntraID" src="https://github.com/user-attachments/assets/37424310-a405-4f2b-97fd-1b241966b2f2" />

**Escenario 2: El reviewer “Josue Rivero” no responde a la revisión**

Ante la falta de respuesta, vemos que el estado del access review cambio. paso de tener usuarios en “Not Reviewed” (Sin respuesta) a pasarlos a “Deny” (acceso denegado)

<img width="1583" height="545" alt="02-AccessReview-Overview-EntraIDpng" src="https://github.com/user-attachments/assets/cadb57af-6881-46d6-9c49-d87423f94bef" />
<img width="1591" height="560" alt="02-AccessReview-Deny" src="https://github.com/user-attachments/assets/f9671ef0-d50c-4fc1-be22-e1f4cc5f97ed" />

* **Audit Log Events:** 
**Configuración y creación de PIM** 

El momento en el cual el grupo UserAdmins es vinculado al motor PIM

<img width="580" height="569" alt="Logs-PIM-UserAdmins" src="https://github.com/user-attachments/assets/f4c4282d-b875-4e27-83e4-0e908df9eccf" />

Inicia el proceso para conceder la condición "Elegible" del user "PruebaUserAdmin"

<img width="575" height="565" alt="Logs-PIM-UserElegible" src="https://github.com/user-attachments/assets/33e8ca3e-ba43-4aa5-9e4c-6fcde4a41963" />

Confirmación de que el user "PruebaUserAdmin" puede ser Elegible para integrar el grupo UserAdmins de forma temporal via PIM y JIT

<img width="575" height="567" alt="Logs-UserElegibleOK" src="https://github.com/user-attachments/assets/f97bee47-b2a1-4f3d-bc28-0b174ef4ea22" />

En este log se documentan las configuraciones realizadas al rol "member" dentro del grupo "User Admins" (confguraciones tales como “Duracion Maxima”,”Activacion con MFA” y aprobación via user “Josue Rivero”)

<img width="568" height="654" alt="Logs-ConfiguracionesPIM" src="https://github.com/user-attachments/assets/666a144b-19b3-4bc0-a186-b5bfc62d2afe" />

**Logs de Ingreso Vía JIT**

Los registros de auditoría de PIM for Groups se emiten secuencialmente según la finalización de transacciones en el backend de Microsoft Entra:

**Acceso Concedido**

Vemos la aprobación del user "Josué Rivero" 

<img width="574" height="581" alt="LOGS-SolicitudJITAprobada" src="https://github.com/user-attachments/assets/ebae6ecb-0fd8-41d6-9ecf-68572f314c5b" />

Tenemos el ingreso exitoso del user PruebaUserAdmin al grupo UserAdmins vía JIT. ademas de detallar la justificación de la solicitud de ingreso

<img width="571" height="604" alt="Logs-IngresoJITok" src="https://github.com/user-attachments/assets/f6fb5e2f-12b8-40e2-85f4-14c7c7d7f247" />

**Acceso Vencido** 

Una vez cumplido el plazo de acceso vía JIT, de forma automatizada se revocaron los accesos del user al grupo UserAdmins

<img width="579" height="569" alt="Logs-JIT-AccesoVencido" src="https://github.com/user-attachments/assets/92f1f731-aa76-45e1-9b71-77f89974883b" />

**Access Review** 

Log de Creación del Access Review

<img width="484" height="471" alt="Logs-AccessReview-Create" src="https://github.com/user-attachments/assets/8993fff4-1ab9-43bf-bb5f-b6e1a9ddff34" />

Log del **Escenario 1** en cual el reviewer aprueba el usuario

<img width="840" height="373" alt="Logs-AccessReview-Approve" src="https://github.com/user-attachments/assets/af987c1d-b32f-476b-b658-bd48e4af5876" />

Vemos los logs del **Escenario 2**. en el cual el reviewer no da respuesta a la revisión 

Vemos como termino el tiempo de la revisión 

<img width="839" height="465" alt="Logs-AccessReview-Ended" src="https://github.com/user-attachments/assets/4aa48352-d712-4329-b4bf-0a447a200ab3" />

Vemos como al no tener respuesta del reviewer se activa el Auto-Review para evaluar las acciones a tomar en base alo que se configuro en este proyecto

<img width="588" height="370" alt="Logs-AccessReview-AutoReview" src="https://github.com/user-attachments/assets/8b6ef123-390c-4afa-97ce-a75ee99be7d5" />

vemos que los accesos del usuario "PruebaUserAdmin" son revocados

<img width="756" height="370" alt="Logs-AccessReview-Remove" src="https://github.com/user-attachments/assets/b6bb22e7-249d-4c59-9445-1d57d7019cb1" />

Log final que notifica y confirma la eliminación de los accesos del user de forma automatizada  

<img width="464" height="422" alt="Logs-AccessReview-AutoApplyReview" src="https://github.com/user-attachments/assets/2d7628b6-68e7-4f6f-9ab0-bae5b379a397" />

—

 ## CONCLUSIONES 
 
### **Mitigación efectiva de la acumulación de permisos (Privilege Creep)**
La prohibición de asignaciones permanentes (tanto activas como elegibles) y la caducidad estricta de las membresías, bloquean de raíz el fenómeno del privilege creep. reduciendo drásticamente la superficie de ataque ante la potencial vulneración de credenciales
### **Adopción exitosa de Zero Standing Access (ZSA) vía PIM for Groups**
El uso de grupos habilitados para roles integrados con PIM, además del uso obligatorio de activaciones JIT, sujetas a ventanas temporales acotadas y la necesidad de justificar la solicitud de acceso garantiza necesidad legítima de privilegios elevados.
Con el requerimiento de MFA y un flujo de doble validación (aprobación del User Josue Rivero) Se garantiza la trazabilidad.
### **Resiliencia operacional ante la inacción (Fail-Safe Access Reviews)**
Las campaña de Access Reviews automatizada configurada para que en caso de inacción de parte del reviewer elimine los accesos del user traslada la responsabilidad de la seguridad de pasiva a activa.Evitando el riesgo crítico de accesos huérfanos, La aplicación automática de resultados (Auto-apply results) garantiza que el estado real del directorio coincida permanentemente con las políticas de gobernanza y cumpliendo con marcos regulatorios internacionales 
### **Trazabilidad forense y auditabilidad integral**
Los registros de auditoría (Audit Logs) analizados durante el ciclo de vida de PIM (desde la solicitud elegible hasta la revocación automática por expiración temporal) y los eventos de cierre del Access Review, proporcionan una evidencia técnica incuestionable. Garantizando un historial forense transparente que permite demostrar y responder dudas sobre lo ocurrido en este proyecto

