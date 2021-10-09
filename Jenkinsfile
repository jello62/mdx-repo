pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        script {
          parentPom = readMavenPom file: "mdx-portal-test/pom.xml"

          // add version number to build displayName
          BUILD_VERSION="${RELEASE_VERSION}.${currentBuild.number}"
          currentBuild.displayName = "${BRANCH}-${BUILD_VERSION}"

          dir('mdx-portal-test') {
            bat 'mvn -v'
            bat 'mvn clean install'
          }
        }

      }
    }

    stage('Artifact Upload') {
      steps {
        script {
          if (parentPom.version.contains("-SNAPSHOT")) {
            nexusRepository = "${NEXUS_SNAPSHOTS_REPOSITORY}"
          } else {
            nexusRepository = "${NEXUS_RELEASES_REPOSITORY}"
          }

          // upload each module artifact using its pom.xml
          parentPom.modules.each {
            dir("mdx-portal-test/${it}") {
              pom = readMavenPom file: "pom.xml";
              filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
              echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
              artifactPath = filesByGlob[0].path;
              artifactExists = fileExists artifactPath;
              if(artifactExists) {
                echo "*** File: ${artifactPath}, group: ${parentPom.groupId}, packaging: ${pom.packaging}, version ${parentPom.version}";
                /*
                nexusArtifactUploader(
                  nexusVersion: NEXUS_VERSION,
                  protocol: NEXUS_PROTOCOL,
                  nexusUrl: NEXUS_URL,
                  groupId: parentPom.groupId,
                  version: BUILD_VERSION,
                  repository: nexusRepository,
                  credentialsId: NEXUS_CREDENTIAL_ID,
                  artifacts: [
                    [artifactId: pom.artifactId,
                    classifier: '',
                    file: artifactPath,
                    type: pom.packaging],
                    [artifactId: pom.artifactId,
                    classifier: '',
                    file: "pom.xml",
                    type: "pom"]
                  ]
                );
                */
                nexusPublisher nexusInstanceId: 'localhost',
                nexusRepositoryId: 'maven-releases',
                packages: [[$class: 'MavenPackage',
                mavenAssetList: [[classifier: '',
                extension: '',
                filePath: artifactPath]],
                mavenCoordinate: [artifactId: pom.artifactId,
                groupId: parentPom.groupId,
                packaging: pom.packaging,
                version: BUILD_VERSION]]]
              } else {
                error "*** File: ${artifactPath}, could not be found";
              }
            }
          }
        }

      }
    }

    stage('Deploy') {
      steps {
        echo "deploying ${RELEASE_VERSION}"
      }
    }

  }
  tools {
    maven 'Maven'
  }
  environment {
    NEXUS_VERSION = 'nexus3'
    NEXUS_PROTOCOL = 'http'
    NEXUS_URL = 'localhost:8081'
    NEXUS_CREDENTIAL_ID = 'nexus-cred'
    NEXUS_SNAPSHOTS_REPOSITORY = 'maven-snapshots'
    NEXUS_RELEASES_REPOSITORY = 'maven-releases'
    parentPom = ''
    BUILD_VERSION = ''
  }
  parameters {
    string(defaultValue: '2.0.0', description: 'Release version from pom file.', name: 'RELEASE_VERSION', trim: true)
    gitParameter(branch: '', branchFilter: 'origin/(.*)', defaultValue: 'master', description: 'Select branch to build', name: 'BRANCH', quickFilterEnabled: false, selectedValue: 'DEFAULT', sortMode: 'DESCENDING_SMART', tagFilter: '*', type: 'PT_BRANCH')
  }
}