# API RESTful para Gestión de Proyectos

Esta es una API RESTful desarrollada en Go para la gestión de proyectos. Incluye funcionalidades para listar, agregar, eliminar y consultar proyectos almacenados en una base de datos MySQL. Este proyecto está desplegado en un clúster local de Kubernetes utilizando Minikube.

## Características

- CRUD completo para gestionar proyectos.
- Conexión con una base de datos MySQL.
- Desplegado en Kubernetes con 5 réplicas.

---

## Tecnologías utilizadas

- **Lenguaje**: Go
- **Base de datos**: MySQL
- **Clúster**: Kubernetes (Minikube)
- **Orquestador**: kubectl
- **Cliente HTTP para pruebas**: Postman

---

## Pasos realizados para su funcionamiento

### Instalacion  de kubectl

Se ejecuta el siguiente comando para la instalacion de kubectl, este se ejecuta en un cmd como administrador

![image](https://github.com/user-attachments/assets/6a9a0506-b32b-4c65-bb0b-66aefd4a73ce)

Una vez instalado procedemos a verificar la instalacion

![image](https://github.com/user-attachments/assets/dc45b2a6-99a4-4eeb-857b-3c319ab089e9)

### Instalacion de minikube

Se ejecutan los siguientes comando para su instalacion

![image](https://github.com/user-attachments/assets/aa3aedf1-f65a-400a-a2aa-3517401bc7aa)

![image](https://github.com/user-attachments/assets/d5a59b80-504f-4527-83a4-24beff11c0d7)

Una vez instalado, iniciamos el servicio de minikube y verificamos su status

![image](https://github.com/user-attachments/assets/efd29285-93e5-400f-9a75-f005cae3294b)

![image](https://github.com/user-attachments/assets/e0e3e4d4-d299-465a-91e9-7d558ce62816)

---

## Creacion de recursos necesarios para el despligue la api en kubernets

### Creacion de instancia MySQL para el guardado de la informacion

La api tendra una conexion a una base de datos MySQL, para esto necesitamos crear y desplegar un servicio o instancia de mysql. Creamos un archivo .yaml, que sera el encargado de desplegar el servicio de MySQL

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:5.7
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "290200Carlos$"  # Contraseña de root de MySQL
            - name: MYSQL_DATABASE
              value: "residencias"  # Base de datos por defecto
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
  clusterIP: None  # Configuración de servicio para comunicación dentro del clúster

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Una vez creado, creamos nuestro despliego en kubectl con el siguiente comando

![image](https://github.com/user-attachments/assets/40d1f95c-4e39-42b7-8869-95a6e57ac84b)

### Creacion de la imagen de docker

Para poder desplegar nuestra api, tendremos que crear un docker y lo subiremos a docker hub para que al momento de desplegar nuestra api, kubectl pueda reconocer nuestra imagen creada. Creamos primero nuestro dockerfile

```
FROM golang:1.23.2

WORKDIR /app

COPY go.mod ./
COPY go.sum ./

RUN go mod download

COPY *go ./

RUN go build

EXPOSE 8080

CMD ["/app/RestApiPract1"]
```

Despues lo construimos y lo subimos a docker hub como se muestra en las imagenes

![image](https://github.com/user-attachments/assets/8d0d7741-ce24-447d-adb2-61764d2aa742)

![image](https://github.com/user-attachments/assets/1cef7aa7-3062-401e-8e08-40be814dd4eb)

Con esto concluimos con los requerimientos necesarios para desplegar nuestra api.

---

## Despliegue de la Api en kubernets

### Creacion de archivo para despliegue

COmo mencionamos anteriormente necesitamos un archivo yaml, el cual tendra la configuracion para la creacion del despliegue en kubernets, una vez creado quedara de la siguiente forma, teniendo en cuentra las 5 replicas solicitadas

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      app: my-api
  template:
    metadata:
      labels:
        app: my-api
    spec:
      containers:
        - name: my-api-container
          image: carloshmv/my_api
          ports:
            - containerPort: 8080
          env:
            - name: DB_HOST
              value: "mysql-service"
            - name: DB_PORT
              value: "3306"
            - name: DB_USER
              value: "root"
            - name: DB_PASSWORD
              value: "290200Carlos$"
            - name: DB_NAME
              value: "residencias"

---
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: my-api
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: LoadBalancer

```

Una vez creado, desplegamos nuestra api con el siguiente comando

![image](https://github.com/user-attachments/assets/70268a70-96ec-418e-9146-713c884153e9)

Verficamos que esta funcione con el comando kubectl get pods, observando nuestra 5 replicas de neustra api y nuestro servicio de MySQL, concluyendo con nuestro despliegue 

![image](https://github.com/user-attachments/assets/8b8f4dff-fc46-4a91-aa02-f234e8434c97)

---

## Pruebas

Para poder realizar pruebas, necesitamos conocer nuestra url con el siguiente comando

![image](https://github.com/user-attachments/assets/c92afdea-5868-4903-ad3f-3aefc3c85c07)

Una vez descubierta nuestra url procedemos a realizar pruebas con postman, realizando peticiones a nuestros endpoints

### Obtener todos los proyectos

![image](https://github.com/user-attachments/assets/e75538bf-f5b6-427d-9016-88faea2dfec8)

### Agregar proyecto

![image](https://github.com/user-attachments/assets/c20dbe10-17b4-40c2-9942-7aba6f2fb857)

![image](https://github.com/user-attachments/assets/64cfba2c-66c9-41c4-9c67-419b1206d000)

### Ver proyecto por numero de control

![image](https://github.com/user-attachments/assets/06da3fd1-c44c-4489-bd46-37841b6dfd6c)

### Eliminar proyecto

![image](https://github.com/user-attachments/assets/8df90587-991a-4e23-9a3d-a5b98a532b0f)

![image](https://github.com/user-attachments/assets/87eb8da4-e3cc-4d04-9c8a-e72cb8deae01)

---

## Participantes en la creacion del repositorio

- Carlos Humberto Valadez Molina
- Mauricio de Jesus Cardona Ramirez
- Chistian Ernesto Silva Pedraza














