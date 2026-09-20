# Ejemplo 1 - Gestionand un usuario sencillo

El objetivo es crear un nuevo usuario en nuestro cluster, registrarlo y poder comprobar como funciona el RBAC con este usuario.

Como estamos usando un entorno local como minikube, no es necesario firmar el certificado usando el CSR (CertificateSigningRequest) del cluster, podemos hacerlo directamente con OpenSSL usando la CA de Minikube.


1. Let's create a new namespace

> kubectl create namespace office

2. Create a new key for the user (Creamos la clave privada del nuevo usuario)

> openssl genrsa -out employee.key 2048


3. Create the certification request (Creamos la solicitud indicando el usuario y grupo)

> openssl req -new -key employee.key -out employee.csr -subj "/CN=employee/O=mysupercompany"

More info:
- CN -> Common name (this is mapped in k8s as userername)
- O -> organization (This is mapped later into the group)


4. Sign the previuous certificate using the minikube's CA

```sh
openssl x509 -req \
     -in employee.csr \
     -CA ${PATH_CRT}/ca.crt -CAkey ${PATH_CRT}/ca.key \
     -CAcreateserial \
     -out employee-local.crt \
     -days 365

```

- Esto genera el certificado employee-local.crt válido por 1 año

5. Register the new creds and the context in the cluster


Nota:
- La flag  `--embed-certs` permite almacenar en YAML nuestra key y cert, lo que nos evita tener que tener archivos sensibles desperdigados (los podemos eliminar después).  

```sh
# 1. Añadir el usuario a la configuración de kubectl
kubectl config set-credentials employee \
  --client-certificate=employee-local.crt \
  --client-key=employee.key \
  --embed-certs=true

# 2. Crear el contexto vinculando el clúster de Minikube con el nuevo usuario
kubectl config set-context usuario-local-context \
  --cluster=minikube \
  --user=employee \
  --namespace=office
```

We can list now the new context:

> kubectl config get-contexts


If need to delete the context `kubectl config delete-context  <context-name> ` 



6. Change to new user context

> kubectl config use-context usuario-local-context


If we try to list the pods in the current context we might see this message below, as we did not add any Role and binding yet:

> Error from server (Forbidden): pods is forbidden: User "employee" cannot list resource "pods" in API group "" in the namespace "office"

