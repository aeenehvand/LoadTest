pipeline {
  agent any
  options { timestamps() }

  stages {
    stage('Run JMeter') {
      steps {
        sh '''
          set -euxo pipefail

          # Find JMeter on this Mac (works for Homebrew on Apple Silicon/Intel)
          JMETER="$(command -v jmeter || true)"
          if [ -z "$JMETER" ]; then
            if [ -x /opt/homebrew/bin/jmeter ]; then JMETER=/opt/homebrew/bin/jmeter
            elif [ -x /usr/local/bin/jmeter ]; then JMETER=/usr/local/bin/jmeter
            else
              echo "JMeter not found on agent PATH. Please install it (brew install jmeter)."
              exit 1
            fi
          fi

          rm -rf report results.jtl || true
          mkdir -p report
          "$JMETER" -n -t example_test.jmx -l results.jtl -e -o report
        '''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'results.jtl, report/**', fingerprint: true
      script {
        try {
          publishHTML(target: [
            reportDir: 'report',
            reportFiles: 'index.html',
            reportName: 'JMeter Report',
            keepAll: true,
            alwaysLinkToLastBuild: true
          ])
        } catch (ignored) {
          echo 'HTML Publisher plugin not installed — artifacts archived only.'
        }
      }
    }
  }
}
