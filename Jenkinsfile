pipeline {
  agent { label 'ubuntu-18.04' }

  environment {
    PROFILE_x86_64 = 'clang-6.0-linux-x86_64'
    PROFILE_x86 = 'clang-6.0-linux-x86'
    CPUS = """${sh(returnStdout: true, script: 'nproc')}"""
    CC = 'clang-6.0'
    CXX = 'clang++-6.0'
  }

  stages {
    stage('Setup') {
      steps {
        sh 'mkdir -p install'
      }
    }
    stage('Pull Request pipeline') {
      when { changeRequest() }
        stages {
          /*
          stage('Unit tests') {
            steps {
              sh script: "mkdir -p unittests", label: "Setup"
              sh script: "cd unittests; env CC=gcc CXX=g++ cmake ../test/unit", label: "Cmake"
              sh script: "cd unittests; make -j $CPUS", label: "Make"
              sh script: "cd unittests; ctest", label: "Ctest"
            }
          }
          */
          stage('Build mana') {
            steps {
              sh script: "mkdir -p build", label: "Setup"
              sh script: "cd build; conan install .. -pr $PROFILE_x86_64", label: "conan install"
              sh script: "cd build; cmake ..", label: "cmake configure"
      	      sh script: "cd build; make -j $CPUS", label: "Make"
            }
          }
          stage('build example') {
            steps {
            	sh script: "mkdir -p build_example", label: "Setup"
      	      sh script: "cd build_example; conan install ../unit/integration/simple -pr $PROFILE_x86_64", label: "conan_install"
      	      sh script: "cd build_example; source activate.sh; cmake ../unit/integration/simple", label: "cmake configure"
      	      sh script: "cd build_example; source activate.sh; make", label: "build"
            }
          }
        }
      }
      stage('Deploy package pipeline') {
        when {
          anyOf {
            branch 'master'
          }
        }
        stages {
          stage('Build Conan package') {
            steps {
              //TODO foreach profile ?
              build_conan_package("$PROFILE_x86_64")
            }
          }
          stage('Upload to bintray') {
            script {
              def version = sh (
                script: 'conan inspect -a version . | cut -d " " -f 2',
                returnStdout: true
              ).trim()
              sh script: "conan upload --all -r $REMOTE includeos/${version}@$USER/$CHAN", label: "Upload to bintray"
            }
          }
        }
      }
    }
  }
}

def build_conan_package(String profile) {
  sh script: "conan create . $USER/$CHAN -pr ${profile}", label: "Build with profile: $profile"
}
