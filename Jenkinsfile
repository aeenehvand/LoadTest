pipeline {
  agent any
  options { timestamps() }

  parameters {
    string(name: 'THREADS', defaultValue: '5', description: 'Virtual users')
    string(name: 'RAMP',    defaultValue: '5', description: 'Ramp-up seconds')
    string(name: 'LOOPS',   defaultValue: '2', description: 'Loops per user')
  }

  stages {
    stage('Run JMeter') {
      steps {
        sh 'set -euxo pipefail'
        // Find JMeter path and store it
        sh 'if command -v jmeter >/dev/null 2>&1; then echo $(command -v jmeter) > jm_path.txt; elif [ -x /opt/homebrew/bin/jmeter ]; then echo /opt/homebrew/bin/jmeter > jm_path.txt; elif [ -x /usr/local/bin/jmeter ]; then echo /usr/local/bin/jmeter > jm_path.txt; else echo "JMeter not found. Install with: brew install jmeter" ; exit 1; fi'

        sh 'rm -rf report results.jtl || true'
        sh 'mkdir -p report'

        // Run the test
        sh '"$(cat jm_path.txt)" -n -t example_test.jmx -Jthreads="$THREADS" -Jramp="$RAMP" -Jloops="$LOOPS" -l results.jtl'

        // Generate HTML dashboard if not already created
        sh '[ -f report/index.html ] || "$(cat jm_path.txt)" -g results.jtl -o report'

        // Debug listings (helpful if you don’t see report/)
        sh 'echo "=== Workspace ==="; ls -lah; echo "=== report/ ==="; ls -lah report || true'
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'results.jtl, report/**', fingerprint: true
      echo 'Open: Build → Artifacts → report → index.html'
    }
  }
}
