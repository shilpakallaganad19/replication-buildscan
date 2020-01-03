pipeline {
    agent any
    stages {
        stage ('Clone') {
            steps {
                git branch: 'master', url: "https://github.com/jfrog/project-examples.git"
            }
        }

        stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "tata-art",
                    url: "http://34.93.114.225:8081/artifactory",
                    credentialsId: "CREDENTIALS"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "tata-art",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "tata-art",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-snapshot"
                )
            }
        }

        stage ('Exec Maven') {
            steps {
                rtMavenRun (
                    tool: maven, // Tool name from Jenkins configuration
                    pom: 'maven-example/pom.xml',
                    goals: 'clean install -U',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
        }

        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "tata-art"
                )
            }
        }

        stage ('Xray scan') {
            steps {
                xrayScan (
                    serverId: "tata-art",
                    failBuild: false
                )
            }
        }
    }
}
