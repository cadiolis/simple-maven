node {
	def commitId, version

    stage('Prep') {
		checkout scm

		commitId = sh (script: 'git rev-parse HEAD', returnStdout: true)
		echo "commitId: '$commitId'"
		echo "commitId: '${commitId.substring(0,7)}'"
		def commitDate = sh(script: "git show -s --format=%cd --date=format:%Y%m%d-%H%M%S ${commitId}", returnStdout: true).trim()
		echo "commitDate: '$commitDate'"

		def pom = readMavenPom(file: 'pom.xml')
		echo "pom.version: '${pom.version}'"
		version = pom.version.replace("-SNAPSHOT", ".${commitDate}.${commitId.substring(0, 7)}")
		echo "version: '$version'"

		currentBuild.displayName = "#${currentBuild.number} - ${version}"
	}
	stage('Build') {
		withMaven(jdk: 'JDK8u172', maven: 'M3') {
			sh "mvn clean package"
		}
	}
	stage('Creating tag') {
		echo "VERSION: '$version'"
        createTag nexusInstanceId: 'nxrm3', tagAttributesJson: '{"createdBy" : "JohnSmith"}', tagName: "'$version'"
	}
    stage('Example') {
        if (env.BRANCH_NAME == 'master') {
            echo 'I only execute on the master branch'
        } else {
            echo 'I execute elsewhere'
        }
    }
}
