# Variables

Las variables que podemos definir en GitLab son un tipo de variables de entorno.

Se puede utilizar para:

* Controlar el comportamiento de nuestros _pipelines_
* Reutilizar su valor en diferentes lugares
* Evitar el _hard-coding_

Las variables se pueden definir en varios sitios:
* En el fichero `/.gitlab-ci.yml`
* A nivel de proyecto
* A nivel de grupo
* A nivel de instancia

## Variables predefinidas

GitLab pone a nuestra disposición un nutrido listado de 
[variables predefinidas](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html) que podemos utilizar
directamente en nuestros _pipelines_ y scripts.

Ya hemos utilizado estas variables predifinidas en los ejemplos anteriores.

## Rama de trabajo

Para hacer los ejemplos, crearemos una rama a partir de master que denominaremos `variables`:

```bash
$ git checkout -b variables main
```

Todos los commits que especifiquems a continuación los haremos sobre esta rama.

## Configuración previa

Para simplificar los _pipelines_, cambiar la configuración para permitir que el _runner_ que tenemos configurado pueda
ejecutar cualquier trabajo. Para ello, ir a _Settings -> CI/CD -> Runners_, editar el runner y activar la opción
_Run untagged jobs_.


## Definir variables en `/.gitlab-ci.yml`

Para definir una variable utilizamos la clave `variable`:

```yaml
variables:
  NUMBER_OF_ITEMS: 3
  DEFAULT_OUTPUT: "$NUMBER_OF_ITEMS items"
stages:
  - build
  - test
  - build
Usando variables:
  stage: build
  variables:
    DEFAULT_OUTPUT: "$NUMBER_OF_ITEMS elementos"
    COLOR: rojo
  script:
    - echo "$DEFAULT_OUTPUT" de color "$COLOR"
    - echo "$DEFAULT_OUTPUT de color $COLOR"
```

Para poder utilizar el signo `$`, evitando que este se interprete, utilizar `$$`:

```yaml
Doble dolar:
  stage: build
  variables:
    FLAGS: '-al $$HOME'
  script:
    - 'echo El comando sería ls "$FLAGS"'
```

## Definir variables en el proyecto

A través de la interfaz web, podemos definir variables que luego podemos utilizar en nuestros _pipelines_.

* Ir a la página del proyecto
* Ir a _Settings -> CI/CD -> Variables_
* Hacer click sobre _Add variable_
* Crear una variable con los siguientes valores:
  * Key: PHP_TAG
  * Value: 8.0.11-cli
  * Type: Variable
  * Environment: All
  * Protect Variable: NO
  * Mask variable: NO
* Crear el siguiente Dockerfile
  ```Dockerfile
  ARG PHP_TAG
  FROM php:$PHP_TAG
  ```

* Añadir el siguiente trabajo al _pipeline_
  ```yaml
  Construir imagen:
    stage: build
    script:
      - docker build --build-arg PHP_TAG="$PHP_TAG" --tag myphpimage .
  Ejecutar imagen:
    stage: test
    script:
      - docker run --rm myphpimage php -i
  ```

* Hacer commit y push de los cambios
* Ver el resultado de ejecución del _pipeline_ en la página de GitLab

Igual que hemos hecho para añadir una variable a un proyecto, podemos 
[añadir una variable a un grupo](https://docs.gitlab.com/ee/ci/variables/#add-a-cicd-variable-to-a-group)
o [a una instancia](https://docs.gitlab.com/ee/ci/variables/#add-a-cicd-variable-to-an-instance).

## Variables de tipo fichero


* Ir a la página del proyecto
* Ir a _Settings -> CI/CD -> Variables_
* Hacer click sobre _Add variable_
* Crear una variable con los siguientes valores:
  * Key: APP_CONFIG
  * Type: File
  * Value:
    ```bash
    PORT: 5574
    BIND_IP: 0.0.0.0
    ```
  * Environment: All
  * Protect Variable: NO
  * Mask variable: NO
* Añadir el siguiente trabajo a nuestro pipeline

```yaml
Fichero de configuración:
  stage: test
  script:
    - echo Contenido de $APP_CONFIG
    - cat $APP_CONFIG
```
* Hacer commit y push de los cambios
* Ver el resultado de ejecución del _pipeline_ en la página de GitLab

## Pasar variables de un trabajo a otro

Para pasar variables de un trabajo a otro dentro de un pipeline, utilizamos artefactos.

* Añadir los siguientes trabajos al _pipeline_

```yaml
Genera numero de version:
  stage: build
  script:
    - echo "BUILD_VERSION=20210715" >> build.env
  artifacts:
    reports:
      dotenv: build.env

Despliega la imagen:
  stage: deploy
  script:
    - echo "$BUILD_VERSION"  #Salida: '20210715'
  needs:
    - job: 'Genera numero de version'
      artifacts: true
```
* Hacer commit y push de los cambios
* Ver el resultado de ejecución del _pipeline_ en la página de GitLab

En este ejemplo, estamos usando [`needs`](https://docs.gitlab.com/ee/ci/yaml/index.html#artifact-downloads-with-needs). Podríamos
realizar el mismo ejemplo utilizando 
[`dependencies`](https://docs.gitlab.com/ee/ci/yaml/index.html#dependencies)


## Documentación

- [Variables (GitLab docs)](https://docs.gitlab.com/ee/ci/variables/)
- [Predefined variables reference](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)
