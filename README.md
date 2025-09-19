#  Guestbook usando docker y k8s 

Este repositorio contiene un Chart de Helm diseñado para desplegar la aplicación "Guestbook" junto con una base de datos MySQL, garantizando Alta Disponibilidad (HA), Escalabilidad Automática (HPA) y Persistencia de Datos.<br />  
<br />

## ✅ Requisitos del Entorno
Asegúrate de tener instaladas y configuradas las siguientes herramientas:

Git: Para clonar el repositorio.

Docker: Para construir la imagen de la aplicación.

Helm 3: Para gestionar el despliegue del Chart.

Minikube/Kubernetes: Un clúster funcional (se recomienda Minikube para el entorno local).

Herramientas de K8s: kubectl y hey (para pruebas de estrés).  

<br />

## 🚀 Guía de Despliegue Paso a Paso
  Sigue estos pasos para poner en marcha la aplicación.

**Paso 1: Clonar el Repositorio**
  Clona el código del proyecto y navega hasta el directorio del Chart:

```
git clone https://github.com/KeepCodingCloudDevops12/miguel_narvaiz-K8s.git
cd <NOMBRE_DEL_REPOSITORIO>/guestbook-app
```

**Paso 2: Preparar la Imagen Docker (Local)**
  Para que Kubernetes pueda encontrar la imagen de la aplicación, debemos construirla localmente e inyectarla en el entorno Docker de Minikube.

2.1 Conectar el shell al Docker de Minikube:

  Este comando redirige tus comandos docker al demonio de Minikube.

```
eval $(minikube docker-env)
```

2.2 Construir la imagen de la aplicación:

Construye la imagen usando la etiqueta definida en tu values.yaml (ej. myapp/guestbook-app:latest). Asegúrate de tener un Dockerfile en el directorio actual.
```
# Reemplaza 'myapp/guestbook-app:v1.0' con la ruta exacta de tu values.yaml
docker build -t <REPOSITORIO>/guestbook-app:v1.0 .
```

2.3 Desconectar el shell de Docker de Minikube:
```
eval $(minikube docker-env -u)
```

**Paso 3: Iniciar Minikube y Addons**

Si tu clúster no está corriendo o si lo eliminaste, inicia Minikube y habilita los complementos necesarios para el HPA y el acceso externo.

```
minikube start --addons=ingress,metrics-server
```

**Paso 4: Desplegar la Aplicación con Helm**
Instala el Chart. Todos los recursos (Deployment, StatefulSet, HPA, Services, etc.) serán creados:
```
helm install guestbook-release .
```

**Verificación Inicial**

Confirma que los Pods estén iniciando. La base de datos (guestbook-db-0) debe arrancar primero.
```
kubectl get all
```
***NOTA: Espera hasta ver los Pods de la aplicación (guestbook-app-$$$) en estado 1/1 Running.***

**Paso 5: Configurar el Acceso Externo (Ingress)**

Para acceder a la aplicación mediante el hostname configurado (guestbook.local), necesitas mapear la IP de Minikube:

5.1 Obtener la IP de Minikube:

```
minikube ip
```

5.2 Mapear el Host en /etc/hosts:

Edita tu archivo de hosts (requiere permisos de administrador) y agrega una línea usando la IP obtenida en el paso anterior.

```
# Ejemplo de la línea a añadir (usa tu IP real)
sudo nano /etc/hosts
# 192.168.49.2 guestbook.local
```

**Paso 6: Prueba de Escalabilidad Automática (HPA)**
Esta prueba demuestra que tu HPA está configurado correctamente para soportar alta carga.

6.1 Generar Tráfico de Estrés:

Ejecuta el siguiente comando para saturar la CPU de los Pods (debe instalar la herramienta hey si no la tiene).

```
hey -n 20000 -c 500 [http://guestbook.local/](http://guestbook.local/)
```

6.2 Monitorear el Escalado:

Mientras el test de estrés se ejecuta, observa cómo el HPA reacciona al superar el umbral del 50% de CPU:

```
watch kubectl get hpa,pods
```

***Verás que la columna TARGETS subirá por encima del 50%, y la columna REPLICAS aumentará de 2 a 3, 4 o 5.***

## ⚙️ Gestión y Mantenimiento
Actualizar Configuraciones (HPA, Réplicas, etc.)
Para cambiar el umbral de CPU del HPA o el número de réplicas mínimas/máximas, edita el archivo values.yaml y luego actualiza el release:

1. Modificar values.yaml:

```
# Ejemplo: escalar al 60% de CPU
autoscaling:
  targetCPUUtilizationPercentage: 60
```
2. Ejecutar la Actualización
```
helm upgrade guestbook-release .
```

## Detener vs. Eliminar el Entorno

```
Acción	              Comando                            ¿Se conservan los datos?                           Propósito
Detener (Halt)        minikube stop                      ✅ Sí (Persistencia garantizada)                   Pausa el entorno de trabajo sin perder datos ni configuraciones.
Eliminar (Delete)	    minikube delete	                ❌ No (Se borra la VM y sus discos)	              Limpia completamente el sistema para empezar de cero.
```

## Desinstalar la Aplicación
```
helm uninstall guestbook-release
```





