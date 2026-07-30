pipeline {
  agent any

  environment {
    LANG = 'en_US.UTF-8'
    APP_NAME = 'ReactNativeJenkins'
    KEYCHAIN_PASSWORD = credentials('mac-keychain-pw')
    ASC_KEY_ID = credentials('asc-key-id')
    ASC_ISSUER_ID = credentials('asc-issuer-id')
    PATH = "/opt/homebrew/bin:${env.PATH}"
  }

  stages {
    stage('Install JS deps') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Prebuild (generate ios/)') {
      steps {
        sh 'npx expo prebuild --platform ios --clean'
      }
    }

    stage('Pods') {
      steps {
        sh 'cd ios && pod install'
      }
    }

    stage('Unlock keychain') {
      steps {
        sh 'security unlock-keychain -p "$KEYCHAIN_PASSWORD" ~/Library/Keychains/login.keychain-db'
      }
    }

    stage('Archive') {
      steps {
        withCredentials([file(credentialsId: 'asc-api-key', variable: 'ASC_KEY_PATH')]) {
          sh '''
            xcodebuild -workspace ios/${APP_NAME}.xcworkspace \
              -scheme ${APP_NAME} \
              -configuration Release \
              -destination "generic/platform=iOS" \
              -archivePath build/${APP_NAME}.xcarchive \
              archive \
              -allowProvisioningUpdates \
              -authenticationKeyPath "$ASC_KEY_PATH" \
              -authenticationKeyID "$ASC_KEY_ID" \
              -authenticationKeyIssuerID "$ASC_ISSUER_ID"
          '''
        }
      }
    }

    stage('Export IPA') {
      steps {
        withCredentials([file(credentialsId: 'asc-api-key', variable: 'ASC_KEY_PATH')]) {
          sh '''
            xcodebuild -exportArchive \
              -archivePath build/${APP_NAME}.xcarchive \
              -exportOptionsPlist exportOptions.plist \
              -exportPath build/ipa \
              -allowProvisioningUpdates \
              -authenticationKeyPath "$ASC_KEY_PATH" \
              -authenticationKeyID "$ASC_KEY_ID" \
              -authenticationKeyIssuerID "$ASC_ISSUER_ID"
          '''
        }
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: 'build/ipa/*.ipa', fingerprint: true
    }
  }
}