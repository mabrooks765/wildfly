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

      println properties.PAR_ENVIRONMENT
      //openshift.withCluster() {
      //  properties.'OCP_PROJECT'
      //}
    }

    stage('Check registry for changes') {
      DIGEST_FILE = "$WORKSPACE/digest.txt"

      NOTHING_TO_DO = false

      latest_tag = getLatestImageTagName(
                      )
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
