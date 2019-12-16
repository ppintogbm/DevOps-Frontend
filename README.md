# DevOps-Frontend
Repositorio de front end para piloto 3

## Descripción

## Instrucciones
1. Por medio del CLI de openshift, ingresar con usuario con privilegios administrativos
 ```bash
 oc login -u system:admin
 ``` 
2. Creamos un proyecto donde desplegar, de no tener alguno
 ```bash
 oc new-project [nombre_del_proyecto]
 ``` 
3. Brindamos privilegios de acceso al usuario developer, al proyecto
```bash
oc adm policy add-role-to-user admin developer
```
4. Brindamos el contexto de seguridad `privileged` a la cuenta de servicio `jenkins`
```bash
oc adm policy add-scc-to-user privileged -z jenkins
```
5. Brindamos el contexto de seguridad `privileged` a la cuenta de servicio `default`6. Configuramos el pipeline de despliegue del presente repositorio, tomando en cuenta las siguientes variables en el siguiente ejemplo de YAML:
    1. `pipeline-name`: Nombre del recurso BuildConfig a ser creado.
    2. `git-url`: Dirección del git. 
    3. `openshift-current-project`: Nombre del proyecto donde se está desplegando el pipeline.
    4. `registry`: Dirección interna del registro de openshift (predeterminado: 172.30.1.1:5000).
    5. `app`: Prefijo para los diferentes recursos e imágenes a ser generadas (predeterminado: api-calculadora). 
    6. `apiAddress`: Ruta donde consumir el API (Ej.: http://[service-name|hostname]:[port]).

`pipeline-name.yaml`
```yaml
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: [pipeline-name]
spec:
  runPolicy: Serial
  source:
    type: Git
    git:
      uri: [git-url]
      ref: [branch-name]
  strategy:
    jenkinsPipelineStrategy:
      jenkinsfilePath: Jenkinsfile
      env:
      - name: project
        value: [openshift-current-project]
      - name: registry
        value: [registry-url]
      - name: app
        value: [app-prefix]
      - name: apiAddress
        value: [api-address]
    type: JenkinsPipeline
```
7. Inicializamos el pipeline creado
```console
oc start-build [pipeline-name]
```
8. Concluido el despliegue, deberíamos observar el mismo en la consola de openshift o incluso por medio del siguiente comando: 
```console
oc get dc -l app=[app-prefix]
```