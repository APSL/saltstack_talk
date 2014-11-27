# Gestores de configuración

---
# Gestores de configuración

* **Lenguaje declarativo** para especificar **como debería estar** (recetas)
* **Recetas** que el gestor aplica a los servidores con definiciones simples
    * Instalación de paquetes
    * Usuarios
    * Ficheros (con plantillas)
    * Permisos
    * Crons
    * Comandos
    * Servicios
    * **Dependencias entre definiciones**
        
---

# Ventajas de los gestores de configuración

* **Menor tiempo de puesta en marcha** para arquitectura ya desarrollada
* Entornos **replicables** y **coherentes**
    * Igualdad entre desarrollo, pruebas y producción
    * Contingencia
* **Evolución de arquitectura aprovechable** por arquitectura antigua
* Las **recetas se comparten** entre arquitecturas de proyectos
* Existen **recetarios públicos** aprovechables
* **Arquitectura como código**
    * **Versionado** de configuración mediante repositorio
    * **Documentación** de bajo nivel **innecesaria**
    
---

# Opciones de los gestores de configuración

* [Muchos sistemas de configuración de código abierto](http://en.wikipedia.org/wiki/Comparison_of_open-source_configuration_management_software)
* Más utilizados
    * [Puppet](http://puppetlabs.com/)
    * [Chef](https://www.getchef.com/)
    * [Saltstack](http://www.saltstack.com/)
    * [Ansible](http://en.wikipedia.org/wiki/Ansible_(software))

---

# Puppet

* Ruby
* Desde 2005
* Cliente - servidor (puppet-master)
* El cliente pide al servidor que se quiere reconfigurar
    * El cliente recoje cada x minutos recetas y las aplica
    * Ejecución manual del cliente
    * Orquestación con Mcollective
* [Muy buenas recetas ya hechas](https://forge.puppetlabs.com)
* [Presentación mía sobre puppet](https://speakerdeck.com/creantbits/puppet-labs)
* **Muy robusto**, aunque desarrollar módulos propios no es sencillo

---        

# Chef

* Ruby
* Desde 2009 
* Cliente - servidor
* Muy usado por grandes 
    * [AWS opwsworks](http://aws.amazon.com/es/opsworks/)
    * Facebook
* [Introducción](https://docs.getchef.com/chef_overview.html)
* [¿Software Libre?](https://coderanger.net/chef-open-source)


---

# Ansible

* Python
* Desde 2012
* Conexión del servidor a los clientes vía ssh, sin agente
* Más simple
* Se crean recetas multi-host que se aplican con un archivo único de configuración
* [Ejemplo de mongodb](https://github.com/ansible/ansible-examples/tree/master/mongodb )
        
