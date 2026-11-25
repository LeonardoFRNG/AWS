# Actividad: ciclo de vida de un usuario IAM

## Objetivo

Crear, consultar, editar, probar y eliminar un usuario IAM desde la AWS CLI, aplicando el principio de menor privilegio.

## Antes de empezar

- AWS CLI v2 instalada y configurada con tu usuario administrador.
- Corre `aws sts get-caller-identity` y confirma que respondes como tu usuario.
- Ten abierto `info.md`: ahí están los comandos que necesitas.

## Parte 1: Crear

1. Crea un usuario llamado `dev-<tu-nombre>`, por ejemplo `dev-laura`.
2. Lista todos los usuarios de la cuenta y confirma que aparece.
3. Muestra los datos de tu usuario y anota su `Arn`.

**Resultado esperado:** el usuario existe y no tiene ningún permiso.

## Parte 2: Editar

4. Agrégale una etiqueta que indique su área:

   `aws iam tag-user --user-name dev-<tu-nombre> --tags Key=area,Value=desarrollo`

5. Cámbiale el nombre a `lector-<tu-nombre>`:

   `aws iam update-user --user-name dev-<tu-nombre> --new-user-name lector-<tu-nombre>`

6. Vuelve a mostrar los datos del usuario y verifica el nombre nuevo y la etiqueta.

**Resultado esperado:** el usuario conserva su etiqueta, pero su `Arn` cambió.

## Parte 3: Dar permisos

7. Adjúntale la política `AmazonEC2ReadOnlyAccess`.
8. Lista las políticas del usuario y confirma que aparece. -> aws iam list-attached-user-policies --user-name lector-leonardo

Pista: el ARN de las políticas de AWS es `arn:aws:iam::aws:policy/[<NombreDeLaPolítica>]`.

## Parte 4: Probar el menor privilegio

9. Crea una llave de acceso para tu usuario. La clave secreta solo se muestra una vez.
10. Configura un perfil nuevo en tu CLI con esa llave:

    `aws configure --profile lector`

11. Comprueba quién eres con ese perfil:

    `aws sts get-caller-identity --profile lector`

12. Ejecuta una acción **permitida**:

    `aws ec2 describe-instances --profile lector`

13. Ejecuta una acción **no permitida**:

    `aws iam list-users --profile lector`

**Resultado esperado:** el paso 12 responde sin error y el paso 13 devuelve `AccessDenied`.

## Parte 5: Quitar permisos

14. Con tu usuario administrador, quítale la política al usuario.
15. Repite el comando del paso 12 con el perfil `lector`.

**Resultado esperado:** ahora la acción que antes funcionaba también falla.

## Parte 6: Eliminar

IAM no deja borrar un usuario que todavía tiene algo asociado. Hazlo en este orden:

16. Lista las llaves de acceso del usuario y bórralas.
17. Confirma que no le quedan políticas adjuntas.
18. Borra el usuario.
19. Lista los usuarios y confirma que ya no aparece.
20. Elimina el bloque `[lector]` de `~/.aws/credentials` y de `~/.aws/config`.

**Reto:** intenta borrar el usuario antes de borrar su llave y lee el error que aparece.

## Entregable

Un archivo `evidencia.md` con:

- Captura o salida de los pasos 3, 6, 8, 12, 13, 15 y 19.
- La respuesta a las preguntas de reflexión.

**Nunca incluyas la clave secreta de la llave de acceso en la evidencia ni la subas a GitHub.**

## Preguntas de reflexión

1. ¿Qué podía hacer el usuario justo después de crearlo, antes de adjuntarle una política? ¿Por qué?

El usuario no podía realizar operaciones sobre los recursos de AWS porque no tenía ninguna política de permisos asignada. Esto se debe a que IAM aplica la denegación implícita, es decir, toda acción está denegada por defecto hasta que se autorice mediante una política.

2. En el error del paso 13, ¿qué tres datos te dice el mensaje?

El mensaje de error nos indica:
- Usuario: lector-leonardo, quien intentó realizar la operación.
- Acción: iam:ListUsers, que corresponde a listar los usuarios de IAM.
- Motivo: ninguna política de identidad le concede permiso para realizar esa acción.
3. ¿Por qué cambió el Arn al renombrar el usuario?

Porque el ARN contiene el nombre del usuario como parte de su estructura. Al cambiar el nombre de dev-leonardo a lector-leonardo, también cambió su ARN. Sin embargo, su UserId se mantuvo igual porque sigue siendo el mismo usuario.

4. ¿Por qué AWS obliga a borrar las llaves antes de borrar el usuario?

Porque las llaves de acceso son credenciales asociadas directamente al usuario. AWS exige eliminarlas antes de borrar el usuario mediante la CLI para evitar que queden credenciales asociadas a una identidad eliminada y garantizar una eliminación correcta y segura.

5. ¿Qué pasaría si le hubieras dado AdministratorAccess en vez de solo lectura? ¿En qué caso real estaría justificado?

El usuario tendría permisos administrativos sobre prácticamente todos los servicios y recursos de AWS, siempre que no existan otras restricciones. Podría crear, modificar y eliminar recursos, administrar usuarios y realizar acciones que podrían generar costos.

Estaría justificado para un administrador de infraestructura en la nube que necesite gestionar toda la cuenta. Sin embargo, siempre es recomendable aplicar el principio de menor privilegio y otorgar únicamente los permisos necesarios para cumplir con sus responsabilidades.
Conclusión
Con esta actividad aprendí a gestionar el ciclo de vida de un usuario IAM mediante AWS CLI, desde su creación hasta su eliminación. También comprendí la importancia de las políticas de seguridad, el principio de menor privilegio y la diferencia entre autenticación y autorización. Además, pude comprobar de manera práctica cómo AWS permite o deniega acciones dependiendo de los permisos asignados a cada usuario.

## Criterios de evaluación

| Criterio | Cumple |
|---|---|
| El usuario fue creado, editado y eliminado en el orden correcto | |
| Se evidencia una acción permitida y una denegada | |
| La evidencia no expone ninguna clave secreta | |
| No quedaron usuarios ni llaves de la actividad en la cuenta | |
| Las preguntas están respondidas con sus propias palabras | |