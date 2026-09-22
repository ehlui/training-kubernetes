# RBAC

> TODO: Añadir una previa introducción a AUTENTIFICACION EN k8s y sobre AUTORIZACIONES


Kubernetes no tiene en su sistema el concepto de usuario, pero en vez de estos depende de certificados, y solo aceptará certificados firmados por su propia CA.

El RBAC es una API que dispone kubernetes para gestionar accesos tanto para usuarios como aplicaciones mediante **roles** y los **bindings** para asignar estos roles. (Aunque no sea la unica forma que acepta k8s, es la más usada a dia de hoy por su sencillez).

Más documentación sobre autorización y autentificación:

- La [autentificación](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) sería el primer paso para detectar quién accede al clúster (para controlar quién o qué puede acceder)

- La [autorización](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) sería **qué** permisos tiene ese usuario o aplicación sobre **qué** recurso/s.



## Roles

Son los que definen qué permisos se habilitan para un namespace concreto. 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: office
  name: manage-pods
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list"]
```

Si necesitaramos un Role para todo nuestro cluster (que fuera más de un namespace) usaríamos los **ClusterRole**.


## Bindings

Solo con los roles no podríamos dar permisos a estos "usuarios". Para ello hace falta agrupar este role con el usuario o grupo concreto. Para esto son los bindings.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: office
  name: manage-pods
subjects:
  - kind: User
    name: "employee"
    apiGroup: rbac.authorization.k8s.io
roleRef:
    kind: Role
    name: manage-pods
    apiGroup: rbac.authorization.k8s.io
```

## Service Account

Semejante a los usuarios externos que se conectan a nuestro clúster, pero en vez de usar la **kubeconfig** para acceder al cluster usa tokens.

Esto en la practica es igual que un usuario, pero con el objetivo  de dar permisos a una app para que acceda a nuestro cluster. Ya sea para poder usar la api de kubernetes y sacar ciertas metricas, como algun webhook o CICD externo que necesite ser invocado o que invoque desde nuestro cluster. (Mísmo concepto que las SA de GCP)

Estas SA se añaden a las apps que necesitemos que tengan acceso. K8s les  inyectará los certs básicos para poder acceder a él (en pocas palabras, k8s inyecta los certs que un usuario externo usa para conectarse a un clúster)

```yaml
# Ejemplo basico de una definicion de SA
# Imaginemos que el usuario previo creado, el "employee" necesita crear una api de backoffice
apiVersion: v1
kind: ServiceAccount
metadata:
  name: backoffice-api
```

```yaml
# En este pod le adjuntamos la SA para que este mismo pod tenga acceso al cluster.
# Este pod tendrá acceso a la api de kubernetes gracias a la referencia que añadimos
# Pero veremos que los permisos que tiene son limitados porque tiene los default, que son los mínimos
apiVersion: v1
kind: Pod
metadata:
  name: backoffice-api
spec:
  containers:
  - image: nginx
    name: backoffice-api
  serviceAccountName: backoffice-api  #--> referencia de la SA para adjuntarla al pod
```


Podemos añadir roles para las SA igual que para los usuarios, por ejempo para nuestro pod:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: office
  name: backoffice-api
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

Y el binding para que la SA tenga este Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: backoffice-api
  namespace: office
subjects:
- kind: ServiceAccount # ---> Podemos ver la gran diferencia con el User, ya que son diferentes.
  name: backoffice-api
roleRef:
  kind: Role
  name: backoffice-api
  apiGroup: rbac.authorization.k8s.io
```

Ahora podemos hacer que nuestro pod consuma de la api de k8s para listar pods, hacer un watch en el namespace de nuestro usuario. Así podemos controlar bien qué acceso damos tanto a un usuario como a una SA, siguiendo el **least privilege principle** para dar permisos en nuestro clúster.