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
        packages: [[$class                            : 'MavenPackage', mavenAssetList: [[classifier: '', extension:
            '', filePath:
            'target/my-app-1.0.jar']], mavenCoordinate: [artifactId: 'my-app', groupId: 'com.mycompany.app',
                                                         packaging : 'jar', version: '1.0']]],
        tagName: tag

    input 'Deployed to incoming. Promote to staging?'
    moveComponents destination: 'test-maven-staging', nexusInstanceId: 'nxrm3', tagName: tag

    input 'Deployed to staging. Promote to production?'
    moveComponents destination: 'test-maven-production', nexusInstanceId: 'nxrm3', tagName: tag

    /*
    // push raw (manual use of nexus-java-api)
    ServerConfig config = new ServerConfig(new URI('http://localhost:8081'), new Authentication('admin', 'admin123'))
    RepositoryManagerV3Client client = RepositoryManagerV3ClientBuilder.create().withServerConfig(config).build()

    // raw asset
    DefaultAsset rawAsset = new DefaultAsset('my-app-1.0-SNAPSHOT-depshield.tar.gz',
        Files.newInputStream(Paths.get("$WORKSPACE/target/my-app-1.0-SNAPSHOT-depshield.tar.gz")))
    rawAsset.addAttribute('filename', 'my-app-1.0-SNAPSHOT-depshield.tar.gz')

    // create raw component
    DefaultComponent rawComponent = new DefaultComponent('raw')
    rawComponent.addAttribute('directory', '/depshield')
    rawComponent.addAsset(rawAsset)

    client.upload('depshield-raw-incoming', rawComponent, tag)
    */
  }
}
