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
		string(defaultValue: "front-end-calculator", description: "Image Name", name: "image")
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
				//script{
				//	tag = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
				//}
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
					sh "docker build -t ${registry}/${project}/${image}:${tag} ."
					sh 'docker login -u $(whoami) -p $(cat /var/run/secrets/kubernetes.io/serviceaccount/token) ' + registry + '/' + project
					sh "docker push ${registry}/${project}/${image}:${tag}"
				}
			}
		}
		/*stage('Deploy/Update'){
			steps{
				container('origin'){
					script{
						openshift.withCluster(){
							openshift.withProject(){
								def deployment = openshift.selector('dc',[template: 'ace', app: image])
								if(!deployment.exists()){             
              						def model = openshift.process("-f", "oc/template.yaml", "-p", "APPLICATION_NAME=${image}", "-p", "IMAGE_NAME=${image}:latest")
              						openshift.create(model)
              						deployment = openshift.selector('dc',[template: 'ace', app: image])
              					}
								openshift.tag("${image}:${tag}","${image}:latest")
              					//deployment.rollout().latest()
              					def latestVersion = deployment.object().status.latestVersion
								def rc = openshift.selector('rc',"${image}-${latestVersion}")
								timeout(time:1, unit: 'MINUTES'){
									rc.untilEach(1){
										def rcMap = it.object()
										return (rcMap.status.replicas.equals(rcMap.status.readyReplicas))
                					}
              					}
							}
						}
					}
				}
			}
		}*/
	}
}
