# Cuándo ejecutar un trabajo

GitLab CI/CD nos permite ejecutar un trabajo en función del valor de una variable, el tipo de evento o
el tipo de _pipeline_ por ejemplo.

Para ello, podemos utlizar las siguientes instrucciones dentro de la definición del trabajo:
* [`rules`](https://docs.gitlab.com/ee/ci/yaml/index.html#rules)
* [`only`](https://docs.gitlab.com/ee/ci/yaml/index.html#only--except)
* [`except`](https://docs.gitlab.com/ee/ci/yaml/index.html#only--except)

## Condicionales usando  `when`

Podemos añadir la clave `when` a un trabajo para seleccionar cuándo ejecutarlo.

Como ejemplo, vamos a añadir un trabajo al pipeline de la rama `feature-1723` para que se 
ejecute únicamente de forma manual, es decir, que vamos a tener que entrar a la página web para 
poder ejecutarlo.

* Añadir el siguiente trabajo al fichero `/.gitlab-ci.yml`

```yaml
Ejemplo when:
  stage: build
  script:
    - 'echo "$(hostname): ---> Se ejecuta solo cuando el pipeline se lanza manualmente."'
    - pwd
    - ls -a
  tags:
    - nuestrorunner
    - linux
  when: manual
```

* Hacer commit y push de los cambios 
* Esto hará que se lance un _pipeline_ asociado al evento push.
* Acceder a este _pipeline_ y comprobar que el trabajo `Ejemplo when` no se ha ejecutado dentro del paso `build`. El
  trabajo está en espera de que se ejecute manualmente
* Hacer click sobre el icono _play_ para lanzar la ejecución del trabajo

Posibles valores de `when`:

* on\_success (default): Ejecutar este trabajo si todos los trabajos en pasos anteriores se han ejecutado con éxito o tienen
  la clave `allow_failure: true`. Este es el valor por defecto.
* manual: solo ejecuta el trabajo si este se lanza manualmente.
* always: ejecutar siempre el trabajo, independientemente del resultado de los pasos anteriores.
* on\_failure: ejecutar el trabajo cuando al menos un trabajo anterior ha fallado.
* delayed: [retrasar la ejecución del trabajo](https://docs.gitlab.com/ee/ci/jobs/job_control.html#run-a-job-after-a-delay).
* never: Nunca ejecutar el trabajo.

En [este enlace](https://docs.gitlab.com/ee/ci/yaml/index.html#when) podéis acceder a la 
documentación completa de la clave `when`.

## Ejecutando un trabajo en un merge request

Vamos a añadir un nuevo trabajo a nuestro pipeline, el trabajo `MR Open`, que solo se ejecutará cuando se añaden commits 
a un _Merge request_ o se lanza un _pipeline_ manualmente desde un _Merge Request_.

* Añadimos el siguiente fichero al fichero `/.gitlab-ci.yml`:

```yaml
MR Open:
  stage: build
  script:
    - 'echo "$(hostname): ---> build. Solo en el evento merge_request_event"'
  tags:
    - nuestrorunner
    - linux
  rules:
  - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

* Hacemos commit y push de este cambio sobre la rama `feature-1723` que usamos 
  en el [paso anterior](../4-run-a-simple-pipeline/README_es.md).
* Ir a la página del _pipeline_ en la web de GitLab, veremos que este trabajo no se habrá ejecutado.

* Crear un _Merge Request_ para la rama `feature-1723`. En la propia página del _Merge Request_ nos mostrará un enlace al _pipeline_.
* Si hacemos click sobre el enlace, accederemos al _pipeline_ y veremos que sólo se ha ejecutado el trabajo `MR Open`

Podemos lanzar este trabajo desde la página del _Merge Request_:

* Ir a la página del _Merge Request_
* Hacer click sobre la pestaña _Pipelines_
* Hacer click sobre el botón _Run Pipeline_
* Se ejecutará de nuevo el _pipeline_ únicamente con este trabajo.

## Combinando reglas

Las reglas pueden combinarse dentro de la instrucción `rules` añadiendo múltiples instrucciones `if`. 

* Modificamos el trabajo `MR Open` de la siguiente manera (le vamos a cambiar el nombre también):

```yaml
Iniciar Workflow de Zapier:
  stage: build
  script:
    - 'echo "$(hostname): ---> curl https://www.zapier.com/XXX/YYY/ZZZ "'
  tags:
    - nuestrorunner
    - linux
  rules:
  - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  - if: '$CI_PIPELINE_SOURCE == "schedule"'
```

El funcionamiento de estas reglas es el siguiente:
* Si el _pipeline_ es para un merge request, se cumple la primera condición y el trabajo se añade al _pipeline_ del merge request
* Si el _pipeline_ no es para un merge request, la primer condición no se cumple y se evalúa la segunda condición
* Si el _pipeline_ está programado, se cumple la segunda condición y el trabajo se ejecutará como parte del pipeline
* En cualquier otro caso, el trabajo no se ejecutará.

* Hacemos commit y push de estas modificaciones
* Ir a la página de _pipelines_ del proyecto
* Ver que se han lanzado dos pipelines:
  * Uno vinculado al evento `push`, que ejecuta todos los trabajos excepto `Iniciar Workflow de Zapier`
  * Un segundo _pipeline_ que se lanza dentro del contexto del _Merge Request_ y que sólo ejecuta el
    trabajo  `Iniciar Workflow de Zapier`
* En el menú lateral del proyecto, seleccionar _Ci/CD -> Schedules_
* Darle al botón _New schedule_
* Programar el pipeline para que se ejecute dentro de 5 minutos `45 * * * *` y seleccionar la rama `feature-1723`
* Esperar a que se ejecute el pipeline (o darle al botón _play_ que aparece junto al evento programado)
* Acceder al pipeline y ver que se ha ejecutado correctamente el trabajo `Iniciar Workflow de Zapier`

Este trabajo, `Iniciar Workflow de Zapier` no se ejecutará si la rama no tiene un PR asociado. Vamos a verlo:

* Crear una rama `test` en el mismo commit al que apunta la rama `feature-1723`
* Hacer push de la rama: `git push -u origin test`
* Esto hará que se ejecute un pipeline relacionado con el evento `push`
* Ir a la página de los _pipelines_ de nuestro proyecto y buscar aquel que corresponde a la rama test (debería ser el último)
* Ver que no se ha ejecutado el paso `Workflow de Zapier`
* Borrar la rama:

```bash
$ git checkout feature-1723
$ git branch -d test
$ git branch -d -r origin/test
$ git push --delete origin test
```

## Opciones de `rules`

La clave `rules` ([ver documentación en este enlace](https://docs.gitlab.com/ee/ci/yaml/index.html#rules)) 
permite definir reglas utilizando las siguientes claves:
* `if`: añade un trabajo al _pipeline_ si la condición se cumple.
* `changes`: comprueba si se han producido cambios en ficheros para añadir o no el trabajo.
* `exists: comprueba si existen ficheros para añadir el trabajo al pipeline.
* `allow_failure: si su valor es `true`, permite que el trabajo falle sin detener el _pipeline_.
* `variables`: permite definir variables ante determinadas condiciones.
* `when`: puede tomar los mismos valores que `when` cuando lo utilizamos fuera de `rules`, al nivel del trabajo. 
   **No se puede mezclar `when` dentro de `rules` con when a nivel del trabajo**.

El _pipeline_ se ejecutará si:
* Existe una regla `if`, `changes` o `exists` que se cumple y que además contiene `when: on_success` (valor por defecto), `when: delayed` o `when: always`.
* Existe una regla que solo contiene `when: on_success`, `when: delayed` o `when: always`

Veremos a continuación algunos ejemplos de uso de estas palabras clave:

### Ejecutar un trabajo si existe un fichero

* Crear una rama `docker`
* Acceder instalar `docker` en el _runner_ siguiendo [la documentación para la instalación](https://docs.docker.com/engine/install/ubuntu/).
* Una vez instalado:
  * Añadir el usuario `gitlab-runner` al grupo `docker`: `$ adduser gitlab-runner docker`
  * Abrir sesión como `gitlab-runner`: `$ sudo su - gitlab-runner`
  * Ejecutar: `gitlab-runner$ docker run --rm hello-world`
* Añadir el sigiente trabajo a nuestro pipeline:

```yaml
Construir y ejecutar imagen de docker:
  script: 
    - docker build -t my-image:$CI_COMMIT_REF_SLUG .
    - docker image ls
    - docker run --rm my-image:$CI_COMMIT_REF_SLUG 
  tags:
    - nuestrorunner
    - linux
  rules:
    - exists:
        - Dockerfile
```

* Crear un fichero `Dockerfile` con el siguiente contenido:

```Dockerfile
FROM ubuntu:lastest
CMD ["echo","Hola desde docker!!!"]
```
* Hacer commit de ambos ficheros y hacer push de la rama:
```bash
$ git add .
$ git commit -m'[docker] Añade fichero Dockerfile'
$ git push -u origin docker
```

* Ir a la página de _pipelines_ del proyecto y verificar que se ha ejecutado el trabajo correctamente.

### Combinando opciones

Para terminar este ejemplo, veremos cómo podemos combinar estas opciones entre sí:

* En la rama `docker`, modificar el trabajo `Construir y ejecutar imagen de docker`:

```yaml
Construir y ejecutar imagen de docker:
  script: 
    - docker build -t my-image:$CI_COMMIT_REF_SLUG .
    - docker image ls
    - docker run --rm my-image:$CI_COMMIT_REF_SLUG 
  tags:
    - nuestrorunner
    - linux
  rules:
    - if: '$CI_PIPELINE_SOURCE == "push"' 
      changes:
        - Dockerfile
      when: manual
      allow_failure: true
```

* Modificar el fichero `Dockerfile`:

```Dockerfile
FROM ubuntu:lastest
VOLUME /app
CMD ["echo","Hola desde docker!!!"]
```

* Hacer commit y push de los cambios
* Acceder al _pipeline_ en la web de GitLab
* Lanzar manualmente la construcción de la imagen
* Añadir un fichero nuevo: `touch index.php`
* Hacer commit y push del fichero
* Comprobar que en este segundo push, el pipeline no ejecuta el paso de construcción de la imagen de docker

En la documentación de GitLab podéis ver más ejemplos de reglas 
[más complejas](https://docs.gitlab.com/ee/ci/jobs/job_control.html#complex-rules) así como 
recomendaciones para 
[evitar que los _pipelines_ se ejecuten por duplicado](https://docs.gitlab.com/ee/ci/jobs/job_control.html#avoid-duplicate-pipelines).

## Uso de `workflow`

[La clave `workflow`](https://docs.gitlab.com/ee/ci/yaml/index.html#workflow), 
definida a nivel de fichero, nos ayuda a determinar si se debe crear o no un un pipeline.

* Dentro de la rama `docker`, añadir el siguiente fragmento de código al prinicipio del fichero:

```yaml
workflow:
  rules:
    - if: $CI_COMMIT_MESSAGE =~ /DRAFT/
      when: never
    - if: '$CI_PIPELINE_SOURCE == "push"'
```

* Hacer commit y push de este cambio
* Comprobar que el _pipeline_ se ejecuta correctamente
* Añadir un cambio al fichero `index.php`, un comentario por ejemplo
* Hacer commit con el siguiente mensaje: `Probando el uso de DRAFT`
* Ir a la página de _pipelines_ en GitLab y verificar que no se ejecuta

## Documentación:

- [Job Control (GitLab Docs)](https://docs.gitlab.com/ee/ci/jobs/job_control.html)
- [Predefined variables reference (GitLab docs)](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)
- [Keyword reference for the .gitlab-ci.yml file (GitLab docs)](https://docs.gitlab.com/ee/ci/yaml/index.html)
