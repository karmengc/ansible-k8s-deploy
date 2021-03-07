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
# Hello-go



# Mattermost


# K8s Dashboard



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




Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
