Role Name
=========

A brief description of the role goes here.

Requirements
------------

Se ha instalado en el host desde el cual se despliega ansible:

ansible-galaxy collection uninstall community.general

(pendiente comprobar si es realmente necesario)

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.



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



# Mattermost
Despliega el contenedor de Mattermost en un pod y de PostgreSQL en otro pod asociado para almacenar toda su información. Para que ésta sea persistente, 

# K8s Dashboard
Este rol despliega el Dashboard de Kubernetes si además de incluirse en la variable apps\_to\_deploy, establecemos a true la variable kubernetes\_enable\_web\_ui.


# Drupal + MariaDB Cluster


- Para acceder desde cualquier miembro del cluster por ssh a las instancias de mariadb:

$kubectl exec -it mariadb-0 mariadb -ndrupal -- /bin/bash


- Para saber las métricas que se exportan desde mariadb:


$kubectl -n mysql port-forward mariadb-0 9104:9104 &

(Ponemos & al final para que quede en segundo plano el forward)

$curl -s http://localhost:9104/metrics


# Prometheus + Grafana

Los servicios se encontrarán una vez desplegados en:

https://monitoring.cluster.local:30443/

y

https://monitoring.grafana.cluster.local:30443/

Para acceder a Grafana se pueden obtener las credenciales desde cualquiera de los miembros del cluster ejecutando:

$kubectl get secret prometheus-grafana -nmonitoring -o yaml

Y desencriptando los valores de la siguiente manera:

echo "YWRtaW4=" | base64 -d



Example Playbook
----------------

Para llamar a este rol se haría de la siguiente manera:

    - hosts: server
      roles:
         - ansible-k8s-deploy

