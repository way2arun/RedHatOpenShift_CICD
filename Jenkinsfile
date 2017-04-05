node('maven') {
  // define commands
  def mvnCmd = "mvn"

  stage 'build'
    git branch: '${SOURCE_REF}', url: '${SOURCE_URL}'
    sh "${mvnCmd} clean install -DskipTests=true"
  stage 'test'
    sh "${mvnCmd} test"
  stage 'deployInDev'
    sh "rm -rf oc-build && mkdir -p oc-build/deployments"
    sh "cp target/openshift-tasks.war oc-build/deployments/ROOT.war"
    // clean up. keep the image stream
    sh "oc project ${DEV_PROJECT}"
    sh "oc delete bc,dc,svc,route -l application=${APPLICATION_NAME} -n ${DEV_PROJECT}"
    // create build. override the exit code since it complains about exising imagestream
    sh "oc new-build --name=${APPLICATION_NAME} --image-stream=jboss-eap70-openshift --binary=true --labels=application=${APPLICATION_NAME} -n ${DEV_PROJECT} || true"
    // build image
    sh "oc start-build ${APPLICATION_NAME} --from-dir=oc-build --wait=true -n ${DEV_PROJECT}"
    // deploy image
    sh "oc new-app ${APPLICATION_NAME}:latest -n ${DEV_PROJECT}"
    sh "oc expose svc/${APPLICATION_NAME} -n ${DEV_PROJECT}"
}
