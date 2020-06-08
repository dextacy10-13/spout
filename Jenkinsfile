#!/usr/bin/env groovy
/*

Create a CI/CD pipeline
* deploy the above code to a development environment
* run a simple php syntax lint type check on it.
* on success it will then automatically escalate that code to to a staging environment (
    they may have to fork the code from master to their own private for this as they
    don't have branch access on the master they can do this bit manually as its a work around not workflow)
* In the staging environment I want the tests that run from that repos test folder running and reports returning from the tests confirming either good to merge to master or not.
* I would like to see them doing the staging environment deployment to either VM's or LXC containers (not docker if possible)
* using ubuntu linux LTS as the host
* php7.3 as the php version.
*/
pipeline {
    agent any
    stages {
        stage('Develop Feature'){
            // step definitions
            echo 'Do some Linting'
        }
        stage("composer_install") {
            sh 'composer install'
        }
    }


}

def launchVm() {
   sh aws ec2 run-instances --image-id $ami --instance-type $type --ssh-key-name $key
}
