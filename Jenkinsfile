node {
	def commitId

    stage('Prep') {
		checkout scm

		commitId = sh (script: 'git rev-parse HEAD', returnStdout: true)
		def commitDate = sh (script: "git show -s --format=%cd --date=format:%Y%m%d-%H%M%S ${commitId}", returnStdout: true)

		def pom = readMavenPom(file: 'pom.xml')
		version = pom.version.replace("-SNAPSHOT", ".${commitDate}.${commitId.substring(0, 7)}")

		currentBuild.displayName = "#${currentBuild.number} - ${version}"
	}
	stage('Build') {
		withMaven(jdk: 'JDK8u172', maven: 'M3') {
			sh "mvn clean package"
		}
	}
    stage('Example') {
        if (env.BRANCH_NAME == 'master') {
            echo 'I only execute on the master branch'
        } else {
            echo 'I execute elsewhere'
        }
    }
}
