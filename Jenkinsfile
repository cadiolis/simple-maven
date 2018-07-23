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
        createTag nexusInstanceId: 'nxrm3', tagAttributesJson: "{'createdBy' : 'todo', 'createdOn' : '$commitDate'}", tagName: "$tag"
	}
    stage('Example') {
        if (env.BRANCH_NAME == 'master') {
            echo 'I only execute on the master branch'
        } else {
            echo 'I execute elsewhere'
        }
    }
}
