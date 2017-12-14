import groovy.json.*

node {
  try {
    stage('Clone Git') {
    println('Clone git repository')
    checkout([$class: 'GitSCM', branches: [
      [name: '*/master']],
      doGenerateSubmoduleConfigurations: false,
      extensions: [],
      submoduleCfg: [],
      userRemoteConfigs: [
        [credentialsId: '9d29a29f-d4c4-43ea-953a-ffdee220dfcd',
        url: 'https://github.com/mabrooks765/wildfly.git'
        ]
      ]
    ])
    }

    stage('Read in properties file') {

      println('Read in properties file')
      Properties properties = new Properties()
      propFile = "$WORKSPACE/jb7oracleds.properties"

      if(fileExists(propFile)) {
        println('Properties file exists')
        sh "cat ${propFile}"
      }
      else {
        failStage('Properties file does not exist')
      }

      File useFile = new File(propFile)
      properties.load(useFile.newDataInputStream())

      ENVIRONMENT = properties."PAR_ENVIRONMENT"

      openshift.withCluster() {
        OCP_PROJECT = openshift.project()
      }

      /*Set ALWAYS_RUN to a boolean value*/
      if (properties."PAR_ALWAYS_RUN" == "true") {
          ALWAYS_RUN = true
      } else {
          ALWAYS_RUN = false
      }

      if (properties."PAR_POC_EDR_DEPLOY" == "true") {
          POC_EDR_DEPLOY = true
      } else {
          POC_EDR_DEPLOY = false
      }

      if (properties."PAR_NPROD_EDR_DEPLOY" == "true") {
          NPROD_EDR_DEPLOY = true
      } else {
          NPROD_EDR_DEPLOY = false
      }

      if (properties."PAR_PROD_EDR_DEPLOY" == "true") {
          PROD_EDR_DEPLOY = true
      } else {
          PROD_EDR_DEPLOY = false
      }

      /*Set the EDR and IDR host based off the environment variable*/
      if (ENVIRONMENT == "POC") {
          EDR_HOST = properties.PAR_EDR_POC_HOST
          IDR_SRC_HOST = properties.PAR_IDR_POC_HOST
      }
      else if (ENVIRONMENT == "NPROD") {
          EDR_HOST = properties.PAR_EDR_NPROD_HOST
          IDR_SRC_HOST = properties.PAR_IDR_NPROD_HOST
      }
      else if (ENVIRONMENT == "PROD") {
          EDR_HOST = properties.PAR_EDR_PROD_HOST
          IDR_SRC_HOST = properties.PAR_IDR_PROD_HOST
      } else {
          failStage('Need to set proper environment variable')
      }
    }

    /* stage('Check registry for changes') {
      DIGEST_FILE = "$WORKSPACE/digest.txt"

      NOTHING_TO_DO = false

      latest_tag = getLatestImageTagName(
                      IMAGE_REGISTRY: properties.PAR_EDR_PROD_HOST,
                      IMAGE_REPOSITORY: properties.PAR_BUILDER_SRC_IMAGE_REPOSITORY,
                      IMAGE_NAME: properties.PAR_BUILDER_SRC_IMAGE_NAME,
                      IMAGE_TAG: 'latest'
                    )
      if (ALWAYS_RUN) {
        println('DEBUG: ALWAYS_RUN set to true, building image.')
        return
      }

      image_digest = getImageDigest(
                      IMAGE_REGISTRY: properties.PAR_BUILDER_SRC_IMAGE_DOCKER_REGISTRY,
                      IMAGE_REPOSITORY: properties.PAR_BUILDER_SRC_IMAGE_REPOSITORY,
                      IMAGE_NAME: properties.PAR_BUILDER_SRC_IMAGE_NAME,
                      IMAGE_TAG: latest_tag
                    )
      //Check to see if previous digest file exist. Should only hit this the first time the job is ran.
      def previous_digest_exists = fileExists(DIGEST_FILE)
      if (previous_digest_exists) {
          println('DEBUG: Previous Digest exist')
          String previous_digest = sh(returnStdout: true, script: "cat $DIGEST_FILE")
          println ('DEBUG: Previous Digest value: ' + previous_digest)

          //trim strings to remove any whitespace, etc..
          image_digest = image_digest.trim()
          previous_digest =  previous_digest.trim()

          if (previous_digest == image_digest) {
              //Pass job as a sucess but do not continue.
              println('DEBUG: Previous run digest is the same. Exiting pipeline and marking it as a success.')
              NOTHING_TO_DO = true
              return
          } else {
            //Write new digest file out
            println('DEBUG: previous and current digest do NOT match. Writing new digest file. Proceeding with next step in pipeline')
            def digest_file = new File(DIGEST_FILE)
            digest_file.write(image_digest)

            //TODO get list of tags associated with this. Sort then use the highest value as the upstream tag
            /*latest_tag = getLatestImageTag(
                    EDR_HOST: EDR_PROD_HOST,
                    IMAGE_REPOSITORY: BUILDER_SRC_IMAGE_REPOSITORY,
                    IMAGE_NAME: BUILDER_SRC_IMAGE_NAME
                )*/

          }

      } else {
          println('DEBUG: Previous digest does not exist. Writing new digest file. Proceeding with next step in pipeline')
          def digest_file = new File(DIGEST_FILE)
          digest_file.write(image_digest)
          //TODO: See the above if/else block
          /*latest_tag = getLatestImageTag(
                  EDR_HOST: EDR_PROD_HOST,
                  IMAGE_REPOSITORY: BUILDER_SRC_IMAGE_REPOSITORY,
                  IMAGE_NAME: BUILDER_SRC_IMAGE_NAME
              )*/
      }

      println('End of stage')
    }*/


    /*
    Per https://devops.stackexchange.com/questions/885/cleanest-way-to-prematurely-exit-a-jenkins-pipeline-job-as-a-success
    this has to be ran outside of a stage. If the digest is the same there is no need to continue with the builds. Pass the build.
    */
    if (NOTHING_TO_DO) {
        println('DEBUG: digest comparison is true. Nothing to build')
        currentBuild.result = 'SUCCESS'
        slackNotifyMessage('Job completed. No image updates from Red Hat.')
        return
    }

    }


  catch (e) {
        echo "In catch block"
        println(e.toString())
        failStage('Pipeline build failure')

    }

}

def failStage(message) {
    //currentBuild.result = 'FAIL'
    //return
    error(message)
}

def getImageDigest(arg) {

    def response = httpRequest  timeout: 600, customHeaders: [
            [name: 'Accept', value: 'application/vnd.docker.distribution.manifest.v2+json']
        ],
        contentType: 'APPLICATION_JSON', httpMode: 'GET',
        url: "https://${arg.IMAGE_REGISTRY}/v2/${arg.IMAGE_REPOSITORY}/${arg.IMAGE_NAME}/manifests/${arg.IMAGE_TAG}"

    String image_digest = response.headers['Docker-Content-Digest']
    echo "DEBUG: Image Digest: ${image_digest}"

    return image_digest.trim()

}
