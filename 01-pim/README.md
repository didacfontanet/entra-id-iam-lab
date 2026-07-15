# 01 · Privileged Identity Management (PIM)

## Objetivo

Configurar acceso just-in-time a un rol administrativo, en vez de asignación permanente, siguiendo el principio de mínimo privilegio.

## Qué hice

1. Activé Microsoft Entra ID P2 (necesario para PIM) en un tenant propio de prueba.
2. Creé un usuario de prueba (testpim) sin ningún rol asignado por defecto.
3. Asigné el rol **User Administrator** a ese usuario como **Eligible** (no activo permanente) mediante PIM.
4. Simulé la activación del rol desde la perspectiva del usuario: solicitud, motivo documentado, duración limitada a 8 horas.
5. Verifiqué el ciclo completo en el log de auditoría de PIM: asignación, activación y caducidad automática.

## Por qué importa

Un rol administrativo asignado de forma permanente es una superficie de ataque innecesaria — si esa cuenta se compromete, el atacante tiene ese acceso todo el tiempo. Con PIM, el acceso solo existe durante la ventana en la que realmente se necesita, y queda todo registrado: quién lo activó, cuándo, y por qué.

## Capturas

_(Pendiente: subir 4 capturas a la carpeta screenshots/ de este módulo — dashboard PIM, asignación eligible, rol activado, log de auditoría)_
