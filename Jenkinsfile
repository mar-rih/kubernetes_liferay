import java.text.SimpleDateFormat
currentBuild.displayName = new SimpleDateFormat("yy.MM.dd").format(new Date()) + "-" + env.BUILD_NUMBER

def props



def label = "jenkins-slave-${UUID.randomUUID().toString()}"

podTemplate(
   label: label, 
   namespace: "${env.NAMESPACE}", 
   serviceAccount: "build",
   yaml: """

apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kubectl
    image: vfarcic/kubectl
    command: ["sleep"]
    args: ["100000"]
  - name: helm
    image: vfarcic/helm:2.8.2
    command: ["sleep"]
    args: ["100000"]
  - name: docker
    image: docker:latest
    volumeMounts:
      - name: dind-storage
        mountPath: /var/run/docker.sock
      - name: build-config
        mountPath: /etc/config        
    command: ["sleep"]
    args: ["100000"]
  volumes:
    - name: dind-storage
      hostPath:
        path: /var/run/docker.sock
        type: File
    - name: build-config
      configMap:
        name: build-config        
"""
) {
node(label) {
    
    stage("newTenant") {
      echo "Create new tenant enviroment..."  
    }
    stage("build") {
      container("docker") {  
           git(url: "${env.REPOBUILD}", branch: "${env.BRANCH_NAME}", credentialsId: "github_marwan")
        //git "${env.REPO}"
        
        sh "cp /etc/config/build-config.properties ."
        props = readProperties interpolate: true, file: "build-config.properties"
        
        //sh "ls -l liferay_6.2"
        sh """docker image build \
          -t ${env.IMAGE}:${env.TAG_BETA} liferay_6.2/. """
        
        withCredentials([usernamePassword(
          credentialsId: "docker-marwan",
          usernameVariable: "USER",
          passwordVariable: "PASS"
        )])
        {
          sh "docker login -u $USER -p $PASS"
        }
        sh """docker image push \
          ${env.IMAGE}:${env.TAG_BETA}"""
      }
    }
    stage("func-test"){
        try{
            container("helm"){
              git "${env.REPODEPLOY}"
              
              
              sh """helm upgrade \
                ${env.CHART_NAME} \
                liferay -i \
                --namespace ${env.NAMESPACE} \
                --tiller-namespace ${env.NAMESPACE} \
                --set image.repository=${env.IMAGE} \
                --set image.tag=${env.TAG_BETA} \
                --set ingress.host=${env.ADDRESS} \
                --set clientname=${env.CLIENT_NAME}-build """
                }
            container("kubectl") {
              sh """kubectl -n liferay-client10-build \
                rollout status deployment \
                ${env.CHART_NAME}-liferay"""
            }
            //check health http
            // sh """curl http://app.${env.NAMESPACE}.${env.ADDRESS} """
            
        } catch(e) {
          error "Failed functional tests"
        } finally {
            container("helm") {
              sh """helm delete \
                ${env.CHART_NAME} \
                --tiller-namespace liferay-client10-build \
                --purge"""
            }
        }
    }
    stage("release") {
      container("docker") {
            sh """docker pull \
              ${env.IMAGE}:${env.TAG_BETA}"""
            
            sh """docker image tag \
              ${env.IMAGE}:${env.TAG_BETA} \
              ${env.IMAGE}:${env.TAG}"""
            
            sh """docker image tag \
              ${env.IMAGE}:${env.TAG_BETA} \
              ${env.IMAGE}:latest"""
            
            withCredentials([usernamePassword(
               credentialsId: "docker-marwan",
               usernameVariable: "USER",
               passwordVariable: "PASS"
            )]) {
                sh """docker login \
                    -u $USER -p $PASS"""            
            }
            
            sh """docker image push \
                ${env.IMAGE}:${env.TAG}"""
            
            sh """docker image push \
                ${env.IMAGE}:latest"""              
      }
      
    }
    stage("Deploy"){
        try{
            container("helm"){
              sh """helm upgrade \
                ${env.CHART_NAME_PRO} \
                liferay -i \
                --namespace ${env.NAMESPACE_PRO} \
                --tiller-namespace ${env.NAMESPACE} \
                --set image.repository=${env.IMAGE} \
                --set image.tag=${env.TAG} \
                --set ingress.host=${env.ADDRESS} \
                --set clientname=${env.CLIENT_NAME}"""                
            }
            container("kubectl"){
             sh """kubectl -n  ${env.NAMESPACE_PRO} \
                rollout status deployment \
                ${env.CHART_NAME_PRO}-liferay"""
            
            }
        }catch(e){
            container("helm") {
              sh """helm delete \
                ${env.CHART_NAME_PRO} \
                --tiller-namespace liferay-client10-build \
                --purge"""
            
            error "Failed production tests"
            }
        }
      }
    }
  }
