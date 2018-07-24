node {
  def commitId, tag, commitDate

  stage('Prep') {
    checkout scm

    commitId = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
    commitDate =
        sh(script: "git show -s --format=%cd --date=format:%Y%m%d-%H%M%S ${commitId}", returnStdout: true).trim()

    def pom = readMavenPom(file: 'pom.xml')
    def version
    if (pom.version.contains('-SNAPSHOT')) {
      version = pom.version.replace("-SNAPSHOT", ".${commitDate}.${commitId.substring(0, 7)}")
    }
    else {
      version = pom.version + ".${commitDate}.${commitId.substring(0, 7)}"
    }
    tag = "depshield-$version"

    currentBuild.displayName = "#${currentBuild.number} - ${version}"
  }
  stage('Build') {
    withMaven(jdk: 'JDK8u172', maven: 'M3') {
      sh "mvn clean package"
    }
  }
  stage('Creating tag') {
    createTag nexusInstanceId: 'nxrm3', tagAttributesJson: "{'createdBy' : 'todo', 'hash' : '$commitId'}",
        tagName: "$tag"
  }
  stage('Publish') {
    // push jar
    nexusPublisher nexusInstanceId: 'nxrm3', nexusRepositoryId: 'test-maven-incoming',
        packages: [[$class: 'MavenPackage', mavenAssetList: [[ classifier: '', extension:
            '', filePath: 'target/my-app-1.0.jar']],
                    mavenCoordinate: [artifactId: 'my-app', groupId: 'com.mycompany.app',
                                      packaging : 'jar', version: '1.0']]],
        tagName: tag

    // push raw
    // note: would use nexusPublisher... but it doesn't support raw (or anything but Maven) ATM
    // note: would use httpRequest... but doesn't seem to have support for file upload
    def filename = "target/my-app-1.0-depshield.tar.gz"
    nexusArtifactUploader(
        nexusVersion: 'nexus3',
        protocol: 'http',
        nexusUrl: 'localhost:8081',
        groupId: 'depshield',
        repository: 'depshield-raw-incoming',
        credentialsId: 'nxrm3-credentials',
        artifacts: [
            [
             file: filename]
        ]
    )
    /*
    withMaven(jdk: 'JDK8u172', maven: 'M3', mavenSettingsConfig: 'nxrm3-server') {
       sh "mvn wagon:upload-single " +
           "-Dwagon.url=http://localhost:8081/repository/depshield-raw-incoming " +
           "-Dwagon.serverId=nxrm3-server " +
           "-Dwagon.fromFile=$filename " +
           "-Dwagon.toFile=my-app-1.0-depshield.tar.gz"
    }
    */

    md5sum = sh(returnStdout: true, script: "md5sum ${filename} | awk '{ print \$1 }'").trim()
    echo "MD5SUM of $filename: '$md5sum'"

    // Can take a bit for the artifact to show up in search
    retry(10) {
      sleep 1
      // associate raw with tag
      def response = httpRequest url: "http://localhost:8081/service/rest/v1/tags/associate/${tag}?" +
            "repository=depshield-raw-incoming&format=raw&md5=${md5sum}&name=my-app-1.0-depshield.tar.gz",
            authentication: 'nxrm3-credentials',
            httpMode: 'POST',
            validResponseCodes: '200',
            acceptType: 'APPLICATION_JSON',
            contentType: 'APPLICATION_JSON'
        println("Status: " + response.status)
        println("Content: " + response.content)
    }
  }
  stage('Staging') {
    input 'Deployed to incoming. Promote to staging?'

    // Cannot currently use moveComponents since it only allows tag and we need to additionally restrict on format
    // NXRM Core team is working on adding this for us
    //moveComponents destination: 'depshield-raw-staging', nexusInstanceId: 'nxrm3', tagName: tag
    //moveComponents destination: 'test-maven-staging', nexusInstanceId: 'nxrm3', tagName: tag

    // in the meantime, direct rest calls

    // promote raw
    def response = httpRequest url: "http://localhost:8081/service/rest/v1/staging/move/depshield-raw-staging?" +
        "tag=${tag}&format=raw",
        authentication: 'nxrm3-credentials',
        httpMode: 'POST',
        validResponseCodes: '200',
        acceptType: 'APPLICATION_JSON',
        contentType: 'APPLICATION_JSON'
    println("Status: " + response.status)
    println("Content: " + response.content)

    // promote maven (will be docker)
    response = httpRequest url: "http://localhost:8081/service/rest/v1/staging/move/test-maven-staging?" +
        "tag=${tag}&format=maven2",
        authentication: 'nxrm3-credentials',
        httpMode: 'POST',
        validResponseCodes: '200',
        acceptType: 'APPLICATION_JSON',
        contentType: 'APPLICATION_JSON'
    println("Status: " + response.status)
    println("Content: " + response.content)
  }

  stage('Production') {
    input 'Deployed to staging. Promote to production?'

    //moveComponents destination: 'test-maven-production', nexusInstanceId: 'nxrm3', tagName: tag
    // promote raw
    def response = httpRequest url: "http://localhost:8081/service/rest/v1/staging/move/depshield-raw-production?" +
        "tag=${tag}&format=raw",
        authentication: 'nxrm3-credentials',
        httpMode: 'POST',
        validResponseCodes: '200',
        acceptType: 'APPLICATION_JSON',
        contentType: 'APPLICATION_JSON'
    println("Status: " + response.status)
    println("Content: " + response.content)

    // promote maven (will be docker)
    response = httpRequest url: "http://localhost:8081/service/rest/v1/staging/move/test-maven-production?" +
        "tag=${tag}&format=maven2",
        authentication: 'nxrm3-credentials',
        httpMode: 'POST',
        validResponseCodes: '200',
        acceptType: 'APPLICATION_JSON',
        contentType: 'APPLICATION_JSON'
    println("Status: " + response.status)
    println("Content: " + response.content)
  }
}
