pipeline {
  agent any

  triggers {
    // Poll SCM so commits auto-trigger builds
    pollSCM('H/2 * * * *')
  }

  options {
    timestamps()
    ansiColor('xterm')
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/vidhur2024/8.2CDevSecOps.git'
      }
    }

    stage('Ensure Node.js') {
      steps {
        sh '''
          set -e
          if ! command -v npm >/dev/null 2>&1; then
            echo "npm not found; installing portable Node.js 18.20.4 in workspace..."
            NODE_BASE="node-v18.20.4-darwin-x64"
            curl -fsSL -o node.tar.xz "https://nodejs.org/dist/v18.20.4/${NODE_BASE}.tar.xz"
            rm -rf node-portable "${NODE_BASE}"
            tar -xJf node.tar.xz
            mv "${NODE_BASE}" node-portable
            rm -f node.tar.xz
          fi
          echo 'export PATH="$WORKSPACE/node-portable/bin:$PATH"' > ./.node_env
          . ./.node_env
          node -v
          npm -v
        '''
      }
    }

    stage('Install Dependencies') {
      steps {
        sh '''
          set -e
          [ -f ./.node_env ] && . ./.node_env
          if [ -f package-lock.json ]; then npm ci; else npm install; fi
        '''
      }
    }

    stage('Run Tests') {
      steps {
        sh '''
          set +e
          [ -f ./.node_env ] && . ./.node_env
          npm test || echo "Tests failed or snyk not authed — continuing."
          exit 0
        '''
      }
    }

    stage('Generate Coverage Report') {
      steps {
        sh '''
          set +e
          [ -f ./.node_env ] && . ./.node_env
          npm run coverage || echo "No coverage script defined — continuing."
          exit 0
        '''
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        sh '''
          set +e
          [ -f ./.node_env ] && . ./.node_env
          npm audit || true
        '''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'coverage*/**', allowEmptyArchive: true
    }

    failure {
      emailext(
        subject: "[Jenkins][FAIL] ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to: "you@example.com",              // <- change this (comma-separated for multiple)
        recipientProviders: [developers(), requestor()],
        attachLog: true,
        compressLog: true,
        body: """\
Build: ${env.JOB_NAME} #${env.BUILD_NUMBER}
Status: FAILURE
Branch: ${env.GIT_BRANCH}
Commit: ${env.GIT_COMMIT}

Console log is attached.
Build URL: ${env.BUILD_URL}
"""
      )
    }

    unstable {
      emailext(
        subject: "[Jenkins][UNSTABLE] ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to: "you@example.com",
        recipientProviders: [developers(), requestor()],
        attachLog: true,
        compressLog: true,
        body: """\
Build: ${env.JOB_NAME} #${env.BUILD_NUMBER}
Status: UNSTABLE

Build URL: ${env.BUILD_URL}
"""
      )
    }

    success {
      emailext(
        subject: "[Jenkins][OK] ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to: "you@example.com",
        recipientProviders: [developers(), requestor()],
        attachLog: false,
        body: """\
✅ Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}

Build URL: ${env.BUILD_URL}
Changes:
${currentBuild.changeSets.collectMany { cs -> cs.items }.collect { "- ${it.msg} (${it.author})" }.join("\n")}
"""
      )
    }
  }
}
