# Salstack

---

# ¿Porqué Saltstack?

* No solo configuración, apunta a ser una **solución completa**
    * Orquestación, aprovisionamiento, ...
* **Eficiencia**
* **Pythónico** a todos los niveles
    * Escrito en **python**
    * Recetas en formato de definición **YAML**
    * Plantillas en formato **jinja**
* **Módulos propios asequibles**
* **Software libre** y [desarrollándose rápido](http://docs.saltstack.com/en/latest/topics/releases/2014.7.0.html)
* Una empresa a tiempo completo detrás
* [Mucha comunidad](https://github.com/saltstack/salt/pull/18434)

---

# Arquitectura

* **Cliente** (salt-minion) - **servidor multi-maestro** (salt-master)
* Utiliza un **sistema de mensajería** llamado [RAET](https://github.com/saltstack/raet) desarrollado por ellos 
    * Antes ZeroMQ, pero tenía problemas de escalabilidad
* El minion **consume la cola de mensajes**, sin puertos abiertos
    * Periodicamente cada 5 segundos (configurable)
* **Conexión cifrada**, requiere validar la key del cliente en el servidor
* **Minion independiente**, permite **reconfiguración sin servidor**


---

# Conceptos básicos

* Receta
* Estado
* Módulo
* Asociación de minions con recetas
* Datos
    * Grains
    * Pillar


---

# Receta
* Archivo [YAML](http://es.wikipedia.org/wiki/YAML) con la de **definición de estados**
* Puedes **discriminar** en la receta **según datos** (grains o pillar)

# Ejemplo
    !bash
    # /srv/salt/mimysql.sls
    mysql-server:
        pkg:
            - installed
        service.running:
            - name: mysql
            - reload: True
            - require:
                - pkg: mysql-server
            - watch:
                - file: /etc/mysql/conf.d/default.cnf

    {% if pillar['administrado'] == 'apsl' %}
    edu:
        mysql_user.present:
            - host: localhost
            - password: apsl
    {% endif %}
---

# Estado

* Código python que **representa una entidad**
* **Opera** con uno o **varios modulos de ejecución**
* **Lógica intermodular**: discriminación de SO, versiones de servicio, ...
* [Muchos estados de serie](https://github.com/saltstack/salt/tree/develop/salt/states) 
* [Facilidad de hacer uno propio](http://docs.saltstack.com/en/latest/ref/states/writing.html)
* [Ejemplo](https://github.com/saltstack/salt/blob/develop/salt/states/mysql_user.py)
                
---        

# Módulo

* Código python con funcionalidad que opera con el sistema
* [Muchos módulos de serie](https://github.com/saltstack/salt/tree/develop/salt/modules)
* [Facilidad de hacer uno propio](http://docs.saltstack.com/en/latest/ref/states/writing.html#full-state-module-example)
* [Ejemplo](https://github.com/saltstack/salt/blob/develop/salt/modules/mysql.py#L1006)

--- 
# Asociación de minions con recetas

* Se hace mediante un archivo de definición top.sls

# Ejemplo

    !yaml
    # /srv/salt/top.sls
    base:
        'edu.apsl.net':
            - utils

        'web.*':
            - nginx

        'db?.apsl.net':
            - mysql
            - redis
            # /srv/salt/postgresql/init.sls
            - postgresql
            # /srv/salt/postgresql/configure.sls
            - postgresql.configure

---

# Datos, grains

* **Datos de sistema** que se guardan **al iniciar el minion**
    * Sistema operativo
    * Versiones
    * Información de hardware: MAC, IP's
* Puedes guardar tus **datos estáticos** que no se modifiquen
    * Desde /etc/salt/minion

    
--- 

# Datos, pillar

* Diccionario de **datos propios**
* Archivos de definición YAML
* Para pasar datos especificos de servicio o aplicación
* **Reaprovechar recetas**
* **Ruta** de los archivos por defecto: /srv/pillar
* **Asociar datos con hosts**
    * Idem que con recetas: /srv/pillar/top.sls

            
--- 
# Utilidades integradas
* Saltstack se complementa con una serie de utilidades
    * salt-renderer
    * salt-returner
    * salt-runner
    * salt-cloud
    * salt-formulas
    * Peer communication
    

---

# salt-renderer

* **Generar los diccionarios de definición** de otra manera que no sea yml
* Independiente de donde. En top.sls, recetas o pillar. 
* **Varios renderers para escoger**: plantillas jinja, mako, python
* En el archivo de definición lo marcamos con la primera linea (shebang)
* [Ejemplo](http://docs.saltstack.com/en/latest/ref/renderers/all/salt.renderers.py.html)
* Lo utilizamos para traer datos de aplicaciones desde nuestro backend
            
---

# salt-returner

* Por defecto los valores devueltos por los comandos salt son devueltos al master
* **Recoger los resultados devueltos** de las configuraciones para tratarlos o almacenarlos
* Útil para un frontend propio de estadísticas
* [http://docs.saltstack.com/en/latest/ref/returners/](http://docs.saltstack.com/en/latest/ref/returners/)

---

# salt-runner

* Código python **similar a un módulo**
* Pero **se ejecuta siempre en el salt-master**
* Disponibles [runners propios de salt](http://docs.saltstack.com/en/latest/ref/runners/all/index.html#all-salt-runners)
* Interesante **para orquestar ejecución en diferentes hosts**

# Ejemplo
    !python
    # Import salt modules
    import salt.client

    def up():
        '''
        Print a list of all of the minions that are up
        '''
        client = salt.client.LocalClient(__opts__['conf_file'])
        minions = client.cmd('*', 'test.ping', timeout=1)
        for minion in sorted(minions):
            print minion

---

# Ejecución ejemplo

    !bash
    # salt-run test.up
    acan.sandosmx.com
    agenda500-db.dov.apsl.net
    agenda500-worker.dov.apsl.net
    arqueobcn-db.dov.apsl.net
    arqueobcn-worker.dov.apsl.net
    bb02.v5tech.es
    bb03.v5tech.es
    bb04.v5tech.es
    ...

---

# salt-cloud

* Utilidad para provisionar máquinas virtuales
* [Independiente de proveedor](http://salt-cloud.readthedocs.org/en/latest/topics/features.html)
    * Digital Ocean
    * AWS
    * Linode
    * Openstack
* Tras el despliegue 
    * Instala el minion
    * Se configura
    * Se da de alta en el master

---

# salt-formulas

* Recetas compartidas comunitariamente
* [Información oficial de instalación](http://docs.saltstack.com/en/latest/topics/development/conventions/formulas.html)
* **Permite GitFs** desde archivo de config o copiando a disco
* **[Hay muchas preparadas](https://github.com/saltstack-formulas)**
* [Ejemplo](https://github.com/saltstack-formulas/mysql-formula)
* [Solo configuras mediante pillar](https://github.com/saltstack-formulas/mysql-formula/blob/master/pillar.example)
        
--- 

# Peer communication

* Permite **desde un minion ejecutar modulos en otro minion**
* Se definen **permisos** en /etc/salt/master
* **Desde un estado**
* Utilizando el **modulo publish**
* [Más Información](http://docs.saltstack.com/en/latest/ref/peer.html)

---

# Instalación

* Paquete de sistema o vía pip
* Ruta de trabajo: **/srv/salt**
* Configuración
    * /etc/salt/master
    * /etc/salt/minion
* Los minions deben tener configurado el host del salt-master 
    * Por defecto host: salt

---

#  Script de instalación del minion

    !bash
    #!/bin/bash
    ## Script inicial para lanzar en un servidor nuevo
    # Actualiza a ultima version de sistema
    # Configura salt

    sudo apt-get update
    sudo apt-get -y dist-upgrade

    # Instalamos y configuramos el salt-minion con el host master salt.apsl.net
    echo deb http://ppa.launchpad.net/saltstack/salt/ubuntu `lsb_release -sc` main \ 
    | sudo tee /etc/apt/sources.list.d/saltstack.list
    wget -q -O- "http://keyserver.ubuntu.com:11371/pks/lookup?op=get&search=0x4759FA960E27C0A6" \ 
    | sudo apt-key add -
    sudo apt-get update
    sudo apt-get install -y salt-minion
    sudo sed -i 's/\#master\:\ salt/master\:\ salt\.apsl\.net/' /etc/salt/minion
    sudo service salt-minion restart

    sudo apt-get autoremove

---

# Operativa
* *salt-key -L*
    * Listar los minions asociados y por asociar
* *salt-key -a host*
    * Asociar un minion que está pendiente
* *salt-key -d host*
    * Desasociar un minion que está pendiente    
* ***salt mihost state.sls mireceta***
    * Aplicar la receta mireceta.sls a mihost
* ***salt mihost state.highstate***
    * Sincroniza todo custom modules|states, pillar, recetas
    * Aplica recetas del top.sls asociadas a mihost
    
--- 

# Operativa

* *salt -v mihost comando*
    * Para incremetar el verbose del comando
* *salt -l debug mihost comando*
    * Información de debug al ejecutar el comando
* *salt mihost pillar.items*
    * Listar los pillars asociados al host
* *salt mihost grains.items*
    * Listar los grains
        
