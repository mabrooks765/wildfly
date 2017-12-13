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
      propFile = "jb7-oracleds.prop"

      if(fileExists(propFile)) {
        println('Properties file exists')
        sh "cat ${propFile}"
      }
      else {
        failStage('Properties file does not exist')
      }

      File propFile = new File('jb7-oracleds.prop')
      propFile.withInputStream {
        properties.load(it)
      }
    }


  }

  catch (e) {
        echo "In catch block"
        println(e.toString())
        failStage('Pipeline build failure')

    }

}
