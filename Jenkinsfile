#!groovy

node('jenkins-slave-centos') {
    def server = Artifactory.server 'orderbird-artifactory'
    def rtGradle = Artifactory.newGradleBuild()
    def buildInfo = Artifactory.newBuildInfo()

    properties properties: [
      disableConcurrentBuilds()
    ]

    stage ('Clone') {
        cleanWs() // clean the worspace before cloning the repository
        git credentialsId: '94f706af-950d-454d-998e-759409e39179', url: 'git@github.com:orderbird/sample-falcon-technical-foundation.git'
        try{
            sh 'git checkout release/0.x'
        }catch(Exception e){
            error 'No release branch detected. Failing build.'
        }
    }

    stage ('Artifactory / Build Info configuration') {
        rtGradle.tool = 'Gradle421' // Tool name from Jenkins configuration
        rtGradle.usesPlugin = true // Artifactory plugin already defined in build script
        rtGradle.resolver repo:'gradle-resolver-virtual', server: server
        buildInfo.env.capture = true
    }

    stage ('Exec Gradle') {
        sshagent(['94f706af-950d-454d-998e-759409e39179']) {
            def myRelease=params.RELEASE
            if ("${myRelease}" == "candidate" ){
                rtGradle.deployer repo: 'ob-gradle-rc-publisher-local', server: server
                rtGradle.run buildFile: 'build.gradle', tasks: "clean candidate", buildInfo: buildInfo
            }else if ("${myRelease}" == "final"){
                rtGradle.deployer repo: 'ob-gradle-releases-publisher-local', server: server
                rtGradle.run buildFile: 'build.gradle', tasks: "clean final", buildInfo: buildInfo

                sh 'git checkout develop'
                sh 'git merge release/0.x'
                sh 'git push origin develop'

                sh 'git checkout master'
                sh 'git merge release/0.x'
                sh 'git push origin master'

                sh 'git branch -D release/0.x'
                sh 'git push origin --delete release/0.x'
            }

        }
    }

    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }
}
