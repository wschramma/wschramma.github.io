---
layout: post
author: wschramm
title:  "Comandos básicos de GIT"
date:   2020-05-21 15:20:23
categories: [otros]
tags: [git]
---

A continuación se encuentra una lista de comandos básicos usados para diferentes tareas con GIT

**git help**  
Muestra una lista con los comandos más utilizados en GIT

**git init**  
Este comando nos permite crear un repositorio local con GIT, de esta forma ya podremos utilizar git dentro de nuestro proyecto. 
Para ello deberemos estar ubicados dentro de la carpeta de nuestro proyecto y ejecutar el comando.  

Una vez que agregamos archivos y realizamos un commit se creara el branch master por defecto.  

**git clone URL/name.git NombreProyecto**  
Clona un proyecto de git en la carpeta NombreProyecto.

**git status**  
Nos indica el estado del repositorio, por ejemplo cuales están modificados, cuales no están siendo seguidos, etc.

# Añadir archivos

**git add [path]**  
Agrega al repositorio el archivo que le indiquemos.

**git add -A**  
Agrega al repositorio **TODOS** los archivos y carpetas que estén en nuestro proyecto y que GIT no está siguiendo.  

# Commit

**git commit -m "mensaje" [archivos]**  
Hace un commit a los archivos que indiquemos, de esta forma quedaran guardadas nuestras modificaciones.  

**git commit -am "mensaje"**  
Hace commit a los archivos que han sido modificados y que GIT está siguiendo.  

# Branch

**git branch**  
Nos muestra una lista de las ramas existentes en el repositorio.  

**git branch -v**  
Nos muestra una lista de las ramas, mostrando el último commit.

**git branch [branch-name]**  
Crea una nueva rama, clonando la rama desde donde se ejecuto el comando.

**git branch --merged**  
Permite listar las ramas que han sido mezcladas con la actual. Si no tienen un * pueden borrarse, ya que significa que se han incorporado los cambios en la rama actual.

**git branch -d [branch-name]**  
Permite eliminar una rama, si deseamos forzar la eliminación debemos cambiar **-d** por **-D**.

**git checkout [nombre rama]**  
Sirve para cambiar de rama, a la que se le indique (debe existir dentro del repositorio).  

**git merge [branch-name]**  
Realiza un merge entre dos ramas, la dirección del merge seria entre la rama que se indique en el comando, y la rama donde estamos ubicados.  

**git checkout -b [branch-name]**  
Crea una nueva rama clonando la rama desde donde se ejecuto el comando, a su vez cambia de forma automática a la rama creada.  

# STASH

**git stash**  
Guarda el estado en una pila y limpia el directorio para poder cambiar de rama.

**git stash list**  
Muestra la pila.

**git stash apply**  
Vuelve al estado original del directorio. Si aplicamos **stash {n}** podemos especificar un stash en concreto.

**git stash pop**  
Elimina el primer stash en la pila.

# Remotes

**git remote -v**  
Lista los repositorios remotos.

**git remote add [remote-name] [url]**  
Crea un nuevo remoto, es posible descargar el contenido de ese repositorio con **git fech [remote-name]**.  

**git fetch [remote-name]**  
Descarga el contenido de ese repositorio a la máquina local, no sobre escribe nada si GIT esta realizando un seguimiento de esa rama.

**git pull [remote-name] [branch-name]**  
Hace una actualización en nuestra rama local, desde una rama remota que indicamos en el comando. 

**git push [remote-name] [branch-name]**  
Luego de que hicimos un git commit, si estamos trabajando remotamente, este comando sube los archivos al repositorio remoto, específicamente a la rama que indiquemos.

**git push [remote-name] --delete [branch-name]**  
Elimina una rama del remoto.

**git remote show [remote-name]**  
Inspecciona el remote.

**git remote rename [oldname] [new-name]**  
Renombra el remoto, además también renombra las ramas, quedando algo como [new-name]/master.

**git remote rm [remote-name]**  
Permite eliminar el remoto.

# Tag

**git tag**  
Muestra las etiquetas actuales. 

**git show [tag-name]**  
Muestra la información asociada al tag.

**git tag -a [tag-name] -m "mensaje"**  
Crea un nuevo tag.

**git push origin [tag-name]**  
Sube al remoto un tag especifico.

**git push origin --tags**  
Sube al remoto todos los tags.

# Recomendaciones

*Siempre hay que hacer pull antes de push en caso de que alguien haya subido cambios al servidor

*Genera commits pequeños que se centren en resolver un problema, no hagas commits con grandes cambios

*Antes de hacer commit ejecuta **git diff --check** para ver si hemos añadido demasiados espacios que puedan causar problemas en los demás.

  
