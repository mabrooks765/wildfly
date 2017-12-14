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
                      )
    } */


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
