apiVersion: v1
kind: BuildConfig
metadata:
    annotations:
      pipeline.alpha.openshift.io/uses: '[{"name": "jenkins", "namespace": "", "kind": "DeploymentConfig"}]'
    labels:
      app: cicd-pipeline
      name: cicd-pipeline
    name: tasks-pipeline
spec:
    triggers:
      - type: GitHub
        github:
          secret: "secret101"
      - type: Generic
        generic:
          secret: "secret101"
    runPolicy: Serial
    source:
      type: None
    strategy:
      jenkinsPipelineStrategy:
        env:
        - name: DEV_PROJECT
          value: dev-user0
        - name: STAGE_PROJECT
          value: stage-user0
        jenkinsfile: |-
          def version, mvnCmd = "mvn -s configuration/cicd-settings-nexus3.xml"
          pipeline {
            agent {
              label 'maven'
            }
