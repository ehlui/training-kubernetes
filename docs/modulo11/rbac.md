# RBAC

> TODO: Añadir una previa introducción a AUTENTIFICACION EN k8s y sobre AUTORIZACIONES


El RBAC es una API que dispone kubernetes para gestionar accesos tanto para usuarios como aplicaciones mediante **roles** y los **bindings** para asignar estos roles. 

Más documentación sobre autorización y autentificación:


- La [autentificación](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) sería el primer paso para detectar quién accede al clúster (para controlar quién o qué puede acceder)


- La [autorización](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) sería **qué** permisos tiene ese usuario o aplicación sobre **qué** recurso/s.
