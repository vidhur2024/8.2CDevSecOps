# confirm you’re in the repo
git status

# create Jenkinsfile
cat > Jenkinsfile <<'EOF'
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps { git branch: 'main', url: 'https://github.com/vidhur2024/8.2CDevSecOps.git' }
    }
    stage('Install Dependencies') {
      steps { sh 'npm install' }
    }
    stage('Run Tests') {
      steps { sh 'npm test || true' }          // continue even if tests fail
    }
    stage('Generate Coverage Report') {
      steps { sh 'npm run coverage || true' }  // continue if script missing
    }
    stage('NPM Audit (Security Scan)') {
      steps { sh 'npm audit || true' }         // prints CVEs to console
    }
  }
}
EOF

git add Jenkinsfile
git commit -m "Add Jenkins pipeline"
git push
