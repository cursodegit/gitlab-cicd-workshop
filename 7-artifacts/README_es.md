# Artefactos

Conjuntos de archivos o informes, generados por un _pipeline_, a los que GitLab permite acceder a
través de la web.

## Rama de trabajo

Para hacer los ejemplos, crearemos una rama a partir de master que denominaremos `variables`:

```bash
$ git checkout -b artifacts main
```

Todos los commits que especifiquems a continuación los haremos sobre esta rama.

## Configuración previa

Para simplificar los _pipelines_, cambiar la configuración para permitir que el _runner_ que tenemos configurado pueda
ejecutar cualquier trabajo. Para ello, ir a _Settings -> CI/CD -> Runners_, editar el runner y activar la opción
_Run untagged jobs_.

Por último, nos conectaremos al _runner_ por ssh e instalaremos los siguientes paquetes:

```bash
$ apt install zip unzip
```
## Generar un artefacto

* Crear el siguiente script, llamado `compilar.sh` que simula un proceso que genera un fichero `.zip` con nuestra aplicación
```bash
#!/bin/env bash
mkdir build
zip build/app.zip *
```
* Dar permiso de ejecución al fichero:
```
$ chmod +x compilar.sh
```
* Crear el siguiente _pipeline_:

```yaml
stages:
  - build
Compilar:
  stage: build
  script:
    - 'bash compilar.sh'
  artifacts:
    paths:
      - build/app.zip
```
* Hacer commit y push de los cambios
* Ir a la página con los pipelines del proyecto
* Descargar el artefacto:
  * Desde el listado, en el menú contextual del pipeline
  * Accediendo al trabajo y utilizando el enlace para descargar el artefacto

## Borrar un artefacto

* Ir a la página del trabajo que generó el artefacto
* Darle al icono _Erase job log_, situado en la parte superior derecha del log del trabajo (icono de la papelera)
* Esta acción es irreversible.

## Fijar una fecha de expiración para un artefacto

* Modificar el trabajo `Compilar`
```yaml
Compilar:
  stage: build
  script:
    - 'bash compilar.sh'
  artifacts:
    paths:
      - build/app.zip
    expire_in: 2 minutes
```
* Hacer commit y push de los cambios
* Ir a la página del trabajo, después de que este se haya ejecutado, y comprobar que el artefacto
  desaparece pasados dos minutos

## Documentation

- [Job Artifacts (GitLab docs)](https://docs.gitlab.com/ee/ci/pipelines/job_artifacts.html)

