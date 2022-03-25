@Library('CommonLib') _
def repoBuilder = new com.bre.RepoBuilder()
def projectsFolder
def srcGit
def srcGitName
def destProject
def destGit
def multiBranchPipeline = true
def regexValidator = ~'[^_a-zA-Z0-9-\\ ]'



pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: admin-pipeline
  annotations:
    iam.amazonaws.com/role: ${jenkinsSlaveRoleArn}
spec:
  imagePullSecrets:
    - name: artifactory
  containers:
    - name: build-tools
      image: artifactory.bre.mcd.com/docker/bred-build-tools:${buildToolsImageVersion}
      command:
        - cat
      tty: true
  volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  volumes:
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
"""
    }
  }
