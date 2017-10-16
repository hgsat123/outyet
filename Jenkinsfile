node {
  def project = 'sys'
  def appName = 'outyet'
  def feSvcName = "${appName}-frontend"
  def registry = "172.31.17.242:5000"
  def imageNm = "${appName}:${env.BUILD_NUMBER}"
  def imageTag = "${registry}/${project}/${imageNm}"

  checkout scm

  stage 'Build image'
  sh("docker build -t ${imageNm} .")

  stage 'Run Go tests'
  sh("docker run ${imageNm} go test")

  stage 'Push image to registry'
  sh("docker tag ${imageNm} ${imageTag}")
  sh("docker push ${imageTag}")

  stage "Deploy Application"
  // Roll out to production
  // Change deployed image in Production to the one we just built
  sh("sed -i.bak 's#hgsat123/outyet:1.0#${imageTag}#' ./k8s/production/*.yaml")
  def ns_exist = sh("kubectl get ns | grep production | cut -d' ' -f1")
  if(!$ns_exist)
     sh("kubectl create -f ns production") 

  sh("kubectl --namespace=production apply -f k8s/services/")
  sh("kubectl --namespace=production apply -f k8s/production/")
  sh("kubectl --namespace=production apply -f k8s/lb/")
  sh("echo http://`kubectl --namespace=production get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
}
