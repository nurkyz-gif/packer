properties([
    parameters([
        booleanParam(defaultValue: false, description: 'Do you want to run Terraform apply?', name: 'terraform_apply'),
        booleanParam(defaultValue: false, description: 'Do you want to run Terraform destroy?', name: 'terraform_destroy'),
        choice(choices: ['dev', 'qa', 'prod'], description: 'Choose environment: ', name: 'environment'),
        string(defaultValue: '', description: 'Provide AMI NAME', name: 'ami_name', trim: true)
        ])
    ])
node{
    def aws_region_var = ''

    if(params.environment == 'dev'){
        println("Applying for dev")
        aws_region_var = 'us-east-1'
    }
    else if(params.environment == 'qa'){
        println("Applying for qa")
        aws_region_var = 'us-east-2'
    }
    else{
        println("Applying for prod")
        aws_region_var = 'us-west-2'
    }

    def tfvar = """
    s3_bucket = "jenkins-bucket-terraform"
    s3_folder_project = "terraform_ec2"
    s3_folder_region = "us-east-1"
    s3_folder_type = "class"
    s3_tfstate_file = "infrastructure.tfstate"
    
    environment = "${params.environment}"
    region      = "${aws_region_var}"
    public_key  = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCpKIOQAZSNpeyrCL2CqfUj3D5AjcIPTiVB9pI4gjh4cZESIpyyPcCuc8alGjQYlb0zBG1OO4KLyAHgqODSb8LLru0V0MjgnqGQJZYzTWcHYiXpadmxQL8zLJ5ZhM30PBMH18YHoj8QAo0OBN/tWNErkt9fJMrHiFxFjoNVYFl8KulIFphYm3ioSHCWagjBwQDXNJkEiMVTLiz2we699qWmXFLjSjIaFt1Q8lVxAb+7lC15AIgnB0UkDxqDSel0JVjpW8QIlZG11DL4JaXaErDVDJrlIZMJKk6sdYK1rMP7nsYglIo0aVUO9rLoNrAlT/5KwoPrT3dllyh8972GpiTyS1VBdG0SOmKJ2pwTsdBseKrFXQ6UmRr+lQT5g4Rj7dMjCTQWlRHus7em40KmcYpZ/wVWMnDHjybhjry1GM+ycH2yp8JKrmF0pRxTfZsPiVSLh/vpAOKf0FaKk2QtXFg/d1qZayvqYfKwYqHXpUGpqLjv4XR4iVrGn2/a1CrMRc8= nurkyzazhybekova@Nurkyzs-MBP "
    ami_name      = "${params.ami_name}"
    """

    stage("Pull Repo"){
        cleanWs()
        git url: 'https://github.com/ikambarov/terraform-ec2-by-ami-name.git'
        writeFile file: "${params.environment}.tfvars", text: "${tfvar}"
    }

    withCredentials([usernamePassword(credentialsId: 'jenkins-aws-access-key', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
        withEnv(["AWS_REGION=${aws_region_var}"]) {
            stage("Terrraform Init"){
                sh """
                    bash setenv.sh ${params.environment}.tfvars
                    terraform init
                    terraform plan -var-file dev.tfvars
                """
            }        
            
            if(params.terraform_apply){
                stage("Terraform Apply"){
                    sh """
                        terraform apply -var-file ${params.environment}.tfvars -auto-approve
                    """
                }
            }
            else if(params.terraform_destroy){
                stage("Terraform Destroy"){
                    sh """
                        terraform destroy -var-file ${params.environment}.tfvars -auto-approve
                    """
                }
            }
            else {
                stage("Terraform Plan"){
                    sh """
                        terraform plan -var-file ${environment}.tfvars
                    """
                }
            }
        }        
    }    
}
