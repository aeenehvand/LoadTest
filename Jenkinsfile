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
        sh 'if command -v jmeter >/dev/null 2>&1; then echo $(command -v jmeter) > jm_path.txt; elif [ -x /opt/homebrew/bin/jmeter ]; then echo /opt/homebrew/bin/jmeter > jm_path.txt; elif [ -x /usr/local/bin/jmeter ]; then echo /usr/local/bin/jmeter > jm_path.txt; else echo "JMeter not found. Install: brew install jmeter"; exit 1; fi'
        // Clean and run
        sh 'rm -rf report results.jtl || true'
        sh 'mkdir -p report'
        sh '"$(cat jm_path.txt)" -n -t example_test.jmx -Jthreads="$THREADS" -Jramp="$RAMP" -Jloops="$LOOPS" -l results.jtl'
        // If dashboard not produced during run, generate it from JTL
        sh '[ -f report/index.html ] || "$(cat jm_path.txt)" -g results.jtl -o report'
        // Show quick diagnostics
        sh 'echo "=== summary lines from jmeter.log ==="; grep -E "^summary" -n jmeter.log || true'
        sh 'echo "=== JTL line count (incl. header) ==="; wc -l results.jtl || true'
        // Fail fast if JTL has no samples
        sh 'LINES=$(wc -l < results.jtl || echo 0); if [ "$LINES" -le 1 ]; then echo "No samples captured (JTL lines=$LINES). Check Thread Group params in JMX."; exit 1; fi'
        // List files we archived
        sh 'echo "=== Workspace ==="; ls -lah; echo "=== report/ ==="; ls -lah report || true'
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'results.jtl, report/**, jmeter.log', fingerprint: true
      echo 'Open: Build → Artifacts → report → index.html'
    }
  }
}
