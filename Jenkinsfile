pipeline {
    agent any
    tools {
        "org.jenkinsci.plugins.terraform.TerraformInstallation" "terraform-0.12.13"
    }
    parameters {
        string(name: 'WORKSPACE', defaultValue: 'development', description:'setting up workspace for terraform')
    }
    environment {
        TF_HOME = tool('terraform-0.12.13')
        TF_IN_AUTOMATION = "true"
        PATH = "$TF_HOME:$PATH"
        ACCESS_KEY = credentials('AWS_ACCESS_KEY_ID')
        SECRET_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }
    stages {
            stage('TerraformInit'){
            steps {
                dir('ec2_pipeline/'){
                    sh "terraform init -input=false"
                    sh "echo \$PWD"
                    sh "whoami"
                }
            }
        }

        stage('TerraformFormat'){
            steps {
                dir('ec2_pipeline/'){
                    sh "terraform fmt -list=true -write=false -diff=true -check=true"
                }
            }
        }

        stage('TerraformValidate'){
            steps {
                dir('ec2_pipeline/'){
                    sh "terraform validate"
                }
            }
        }

        stage('TerraformPlan'){
            steps {
                dir('ec2_pipeline/'){
                    script {
                        try {
                            sh "terraform workspace new ${params.WORKSPACE}"
                        } catch (err) {
                            sh "terraform workspace select ${params.WORKSPACE}"
                        }
                        sh "terraform plan -var 'access_key=$AWS_ACCESS_KEY_ID' -var 'secret_key=$AWS_SECRET_ACCESS_KEY' \
                        -var-file 'terraform.tfvars' -out terraform.tfplan;echo \$? > status"
                        stash name: "terraform-plan", includes: "terraform.tfplan"
                    }
                }
            }
        }
        stage('TerraformApply'){
            steps {
                script{
                    def apply = false
                    try {
                        input message: 'Can you please confirm the apply', ok: 'Ready to Apply the Config'
                        apply = true
                    } catch (err) {
                        apply = false
                         currentBuild.result = 'UNSTABLE'
                    }
                    if(apply){
                        dir('ec2_pipeline/'){
                            unstash "terraform-plan"
                            sh 'terraform apply terraform.tfplan'
                        }
                    }
                }
            }
        }
    }
}
