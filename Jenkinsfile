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

# Find JMeter on this Mac
JMETER="$(command -v jmeter || true)"
if [ -z "$JMETER" ]; then
  if [ -x /opt/homebrew/bin/jmeter ]; then JMETER=/opt/homebrew/bin/jmeter
  elif [ -x /usr/local/bin/jmeter ]; then JMETER=/usr/local/bin/jmeter
  else
    echo "JMeter not found
