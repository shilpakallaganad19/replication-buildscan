#!/usr/bin/env groovy

properties([[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', numToKeepStr: '7']]])

node {
  def server = Artifactory.server 'Artifactory'
  def rtMaven = Artifactory.newMavenBuild()

    environment {
        MAVEN_HOME = '/usr/share/maven'
    }

  stage('Clone Git Repository') {
    checkout scm
  }

  stage('Set Build Number') {
    props = readProperties file:"/home/tataraov/version_info"
    currentBuild.displayName = "#${currentBuild.number}-${props['BRANCH']}-${props['VERSION']}"
    artifactVersion = "${props['TAG_NAME']}${(props['BRANCH'] != "master" ? "-SNAPSHOT" : "")}".toString()
  }

  stage ('Artifactory configuration') {
    rtMaven.tool = 'maven'
    rtMaven.resolver server: server, releaseRepo: "maven", snapshotRepo: "maven"
    rtMaven.deployer releaseRepo: 'maven-release-local', snapshotRepo: 'maven-snapshot-local', server: server
    rtMaven.deployer.deployArtifacts = false // Disable artifacts deployment during Maven run
    buildInfo = Artifactory.newBuildInfo()
  }

  stage ('Maven Compile') {
    rtMaven.run pom: 'pom.xml', goals: "verify -Dbuild.version="+artifactVersion, buildInfo: buildInfo
  }

  stage ('Deploy') {
    rtMaven.deployer.deployArtifacts buildInfo
  }

  stage ('Publish build info') {
     server.publishBuildInfo buildInfo
  }

  deleteDir()
}
