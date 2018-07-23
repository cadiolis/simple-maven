node {
	def commitId, tag, commitDate

    stage('Prep') {
		checkout scm

		commitId = sh (script: 'git rev-parse HEAD', returnStdout: true)
		commitDate = sh(script: "git show -s --format=%cd --date=format:%Y%m%d-%H%M%S ${commitId}", returnStdout: true).trim()

		def pom = readMavenPom(file: 'pom.xml')
		def version = pom.version.replace("-SNAPSHOT", ".${commitDate}.${commitId.substring(0, 7)}")
		tag = "depshield-$version"

		currentBuild.displayName = "#${currentBuild.number} - ${version}"
	}
	stage('Build') {
		withMaven(jdk: 'JDK8u172', maven: 'M3') {
			sh "mvn clean package"
		}
	}
	stage('Creating tag') {
        createTag nexusInstanceId: 'nxrm3', tagAttributesJson: "{'createdBy' : 'todo', 'hash' : '$commitId'}", tagName: "$tag"
	}
    stage('Publish') {
		nexusPublisher nexusInstanceId: 'nxrm3', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/my-app-1.0.jar']], mavenCoordinate: [artifactId: 'my-app', groupId: 'com.mycompany.app', packaging: 'jar', version: '1.0']]], tagName: tag
    }
}
