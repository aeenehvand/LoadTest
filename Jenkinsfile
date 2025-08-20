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
        sh '''#!/bin/bash
set -euxo pipefail

JMETER="$(command -v jmeter || true)"
if [ -z "$JMETER" ]; then
  if [ -x /opt/homebrew/bin/jmeter ]; then JMETER=/opt/homebrew/bin/jmeter
  elif [ -x /usr/local/bin/jmeter ]; then JMETER=/usr/local/bin/jmeter
  else
    echo "JMeter not found on agent PATH. Install with: brew install jmeter"
    exit 1
  fi
fi

rm -rf report results.jtl || true
mkdir -p report

"$JMETER" -n -t example_test.jmx \
  -Jthreads="$THREADS" -Jramp="$RAMP" -Jloops="$LOOPS" \
  -l results.jtl -e -o report

echo "=== Workspace after run ==="
ls -lah
echo "=== Report dir ==="
ls -lah report || true
'''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'results.jtl, report/**', fingerprint: true
      echo 'Open the HTML via Artifacts → report → index.html'
    }
  }
}
