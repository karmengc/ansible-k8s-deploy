Ansible-k8s-deploy
=========
Este rol despliega los objetos de Kubernetes necesarios para levantar varias aplicaciones que prueban varias de las funcionalidades de la herramienta.


Requirements
------------

Se ha instalado en el host desde el cual se despliega ansible:

ansible-galaxy collection install community.general

(pendiente comprobar si es realmente necesario)


Es necesario haber desplegado previamente ansible-k8s-install para que Kubernetes esté instalado y configurado en las máquinas destino.


Role Variables
--------------

apps\_to\_deploy

kubernetes\_create\_admin\_dashb\_user

kubernetes\_enable\_web\_ui   (Establecida también para ansible-k8s-install)


Aplicaciones para Desplegar
--------------------------
Las aplicaciones a desplegar serán aquellas incluidas en la variable:

```
apps_to_deploy: {'myweb':'8080', 'drupal':'80', 'monitoring':'9090', 'mattermost':'8000', 'monitoring.grafana':'80'}
```

Pueden incluirse todas tal y como se ve en el ejemplo, o sólo añadir los pares nombre/puerto que se deseen. Esta variable es utilizada por el componente Ingress NGINX para habilitar puntos de entrada a las aplicaciones.

Es necesario en tal caso añadir al fichero /etc/hosts del equipo desde el cual se despliegue el rol la siguiente línea para poder acceder a los servicios:

Raspberry Máster:
192.168.1.60 master myweb.cluster.local mattermost.cluster.local dashboard.k8s.ingress.cluster.local monitoring.cluster.local drupal.cluster.local monitoring.grafana.cluster.local

Molecule Máster:
192.168.130.150 master myweb.cluster.local mattermost.cluster.local dashboard.k8s.ingress.cluster.local monitoring.cluster.local drupal.cluster.local monitoring.grafana.cluster.local


Eligiendo según sea el host destino una opción u otra.


# Hello-go
Su finalidad es que a partir de un código en go, se compile, se cree el contenedor en el registro local y se levante un pod con el mismo.
Hay que tener en cuenta que el código de esta aplicación debe de estar en /mnt/nfs/hello-go que es un directorio montado y procedente del host 
desde el cual se despliega, exportado por NFS.

Este despliegue mostrará la URL en la cual se publicará el servicio.


# Mattermost
Despliega el contenedor de Mattermost en un pod y de PostgreSQL en otro pod asociado para almacenar toda su información. Para que ésta sea persistente, se crea
un Persistent Volume que consiste en una unidad NFS que se exporta desde el nodo máster.

Mejora pendiente: En ocasiones no se cargan las imágenes de usuario correctamente en la interfaz web, principalmente cuando el pod se arranca por primera vez,
para solucionarlo provisionalmente se establecen permisos 777 recursivos en la carpeta del Persistent Volume. 
Para evitar esto una posible mejora sería establecer permisos en el arranque del contenedor de Mattermost para que se puedan leer las imágenes.

# K8s Dashboard
Este rol despliega el Dashboard de Kubernetes si establecemos a true la variable kubernetes\_enable\_web\_ui.

Para acceder a la URL:

https://dashboard.k8s.ingress.cluster.local:30443/

Es necesario un token que facilitará este mismo despliegue por pantalla, o bien consultando dentro del cluster:

$ kubectl get secrets -nkubernetes-dashboard

Averiguamos el identificador del secret que guarda el token de admin y hacemos:

$ kubectl get secret admin-user-token-94wvb -nkubernetes-dashboard -o yaml | grep token

Desencriptar ese token con ''base64 -d''

# Drupal + MariaDB Cluster


- Para acceder desde cualquier miembro del cluster por ssh a las instancias de mariadb:

$kubectl exec -it mariadb-0 mariadb -ndrupal -- /bin/bash


- Para saber las métricas que se exportan desde mariadb:


$kubectl -n mysql port-forward mariadb-0 9104:9104 &

(Ponemos & al final para que quede en segundo plano el forward)

$curl -s http://localhost:9104/metrics


Para configurar drupal por primera vez acceder por el NodePort del servicio:

http://drupal.cluster.local:30010/

* Indicar usuario y contraseña:
drupal / Pass1

* base de datos:
drupaldb

* servidor de base de datos:
mariadb 

(servicio headless del clúster de mariadb)


Una vez instalado es posible acceder a:

https://drupal.cluster.local:30443

Para visitar el sitio web.

# Prometheus + Grafana

Los servicios se encontrarán una vez desplegados en:

https://monitoring.cluster.local:30443/

y

https://monitoring.grafana.cluster.local:30443/

Para acceder a Grafana se pueden obtener las credenciales desde cualquiera de los miembros del cluster ejecutando:

$kubectl get secret prometheus-grafana -nmonitoring -o yaml

Y desencriptando los valores de la siguiente manera:

echo "YWRtaW4=" | base64 -d

Se añadido el tag provision\_install\_tags a las tareas de instalación con HELM puesto que actualizaba directamente de versión (haciendo rollin update),
con el fin de controlar actualizaciones no deseadas. 


Example Playbook
----------------

Para llamar a este rol se haría de la siguiente manera:

    - hosts: server
      roles:
         - ansible-k8s-deploy

# Posible timeout
Es posible que al desplegar en el entorno de test la primera vez aparezca un error al reiniciar Kubelet como el siguiente:
```
{"msg": "Timeout (12s) waiting for privilege escalation prompt: "}
```

En tal caso los servicios arrancarán con normalidad al cabo de unos minutos pero no se mostrarán las líneas finales de debug para conocer los
endpoints a los que podemos conectarnos para comprobar el estado de los servicios. En tal caso desplegar de nuevo y aparecerán al final de
manera correcta.

https://github.com/ansible/ansible/issues/14426




