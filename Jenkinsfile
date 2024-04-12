#!/usr/bin/groovy

@Library(['github.com/indigo-dc/jenkins-pipeline-library@1.4.0']) _

pipeline {
    agent {
        label 'docker-build'
    }

    environment {
        dockerhub_repo = "deephdc/deep-oc-federated-server"
        base_cpu_tag = "3.10-bookworm"
    }

    stages {
        stage('Validate metadata') {
            steps {
                checkout scm
                sh 'deep-app-schema-validator metadata.json'
            }
        }
        stage('Docker image building') {
            when {
                anyOf {
                    branch 'main'
                    branch 'tokens'
                    buildingTag()
                }
            }
            steps{

                dir('check_oc_artifact'){
                    // clone checking scripts
                    git url: 'https://github.com/deephdc/deep-check_oc_artifact'
                }

                dir('deep-oc-user_app'){
                    checkout scm
                    script {
                        // build different tags
                        id = "${env.dockerhub_repo}"

                        if (env.BRANCH_NAME == 'main') {
                           // CPU (aka latest, i.e. default)
                           id_cpu = DockerBuild(id,
                                            tag: ['latest'],
                                            build_args: ["tag=${env.base_cpu_tag}",
                                                         "branch=main"])
                        }

                        if (env.BRANCH_NAME == 'tokens') {
                           // CPU
                           id_cpu = DockerBuild(id,
                                            tag: ['tokens'],
                                            build_args: ["tag=${env.base_cpu_tag}",
                                                         "branch=tokens"])
                        }
                    }
                }
            }
            post {
                failure {
                    DockerClean()
                }
            }
        }

        stage('Docker Hub delivery') {
            when {
                anyOf {
                   branch 'main'
                   branch 'tokens'
                   buildingTag()
               }
            }
            steps{
                script {
                    DockerPush(id_cpu)
                }
            }
            post {
                failure {
                    DockerClean()
                }
                always {
                    cleanWs()
                }
            }
        }

        stage("Render metadata on the marketplace") {
            when {
                allOf {
                    branch 'main'
                    changeset 'metadata.json'
                }
            }
            steps {
                script {
                    def job_result = JenkinsBuildJob("Pipeline-as-code/deephdc.github.io/pelican")
                    job_result_url = job_result.absoluteUrl
                }
            }
        }
    }
}
