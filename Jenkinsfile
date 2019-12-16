pipeline{
	agent { 
        	kubernetes{
            		cloud "openshift"
            		label "maven-build"
            		yamlFile "BuildPod.yaml"
		}
    }
	parameters{
		string(defaultValue: "jenkins", description: "Project/Namespace name", name: "project")
		string(defaultValue: "172.30.1.1:5000", description: "Registry",  name:"registry")
		string(defaultValue: "front-end-calculadora", description: "Image Name", name: "app")
		string(description: "API Address", name: "apiAddress")
	}
	stages{
		stage('Prepare'){
			steps{
				container('maven'){
					script{
						tag = sh(  script: "cd CalcExample/;  mvn org.apache.maven.plugins:maven-help-plugin:3.1.0:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true )
						tag = tag.toLowerCase()
					}	
					echo "Tag: ${tag}"
				}
			}
		}
		stage('Build jar'){
			steps{
				container('maven'){
					sh "cd CalcExample/; mvn -B clean package"
				}
			}
   		}
		stage('Docker build'){
			steps{
				container('docker'){
					sh "docker build -t ${registry}/${project}/${app}:${tag} ."
					sh 'docker login -u $(whoami) -p $(cat /var/run/secrets/kubernetes.io/serviceaccount/token) ' + registry + '/' + project
					sh "docker push ${registry}/${project}/${app}:${tag}"
				}
			}
		}
		stage('Deploy/Update'){
			steps{
				container('origin'){
					script{
						openshift.withCluster(){
								openshift.withProject(){
									def deployment = openshift.selector('dc',[template: 'frontend-calculadora', app: app])
									if(!deployment.exists()){             
										def model = openshift.process("-f", "oc/template.yaml", "-p", "APPLICATION_NAME=${app}", "FRONT_IMAGE_NAME=${app}:latest", "-p", "IMAGE_NAMESPACE=${project}", "-p", "API_ADDRESS=${apiAddress}")
										openshift.apply(model)
										deployment = openshift.selector('dc',[template: 'api-calculadora', app: app])
									}
									openshift.tag("${app}:${tag}","${app}:latest")
									/*
									def latestVersion = deployment.object().status.latestVersion
									def rc = openshift.selector('rc',"${app}-${latestVersion}")
									timeout(time:1, unit: 'MINUTES'){
										rc.untilEach(1){
											def rcMap = it.object()
											return (rcMap.status.replicas.equals(rcMap.status.readyReplicas))
										}
									}
									*/
								}
						}
					}
				}
			}
		}
	}
}