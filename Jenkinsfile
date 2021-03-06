podTemplate(label: 'build-agent-go', 
  containers: [
    containerTemplate(name: 'golang', image: 'golang:1.9.3-alpine3.6', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'docker', image: 'docker:17.09', ttyEnabled: true, command: 'cat')
  ],
  volumes: [
    hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
  ]) {
  node('build-agent-go') {
    checkout scm

    stage('Download dependencies') {
      container('golang') {
        sh 'apk add --no-cache git curl'
        sh 'curl -fsSL -o /usr/local/go/bin/dep https://github.com/golang/dep/releases/download/v0.4.1/dep-linux-amd64 && chmod +x /usr/local/go/bin/dep;'
        sh 'export GOPATH="$PWD";cd "src/a10bridge";dep ensure'
      }
    }
    stage('Test application') {
      container('golang') {
        sh 'export GOPATH="$PWD";cd src/a10bridge; go test -v ./...'
      }
    }
    stage('Build application') {
      container('golang') {
        sh 'export GOPATH="$PWD";cd src/a10bridge; CGO_ENABLED=0 GOOS=linux go build -a -tags netgo -ldflags \'-w\' .'
      }
    }
    stage('Build and push docker Image') {
      container('docker') {
        sh 'docker build -t registry.pulsepoint.com/a10bridge src/a10bridge'
        sh 'docker push registry.pulsepoint.com/a10bridge'
      }
    }
  }
}