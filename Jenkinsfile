#!/usr/bin/env groovy
/*

Create a CI/CD pipeline
* deploy the above code to a development environment
* run a simple php syntax lint type check on it.
* on success it will then automatically escalate that code to to a staging environment (
    they may have to fork the code from master to their own private for this as they
    don't have branch access on the master they can do this bit manually as its a work around not workflow)
* In the staging environment I want the tests that run from that repos test folder running
* and reports returning from the tests confirming either good to merge to master or not.
* I would like to see them doing the staging environment deployment to either VM's or LXC containers (not docker if possible)
* using ubuntu linux LTS as the host
* php7.3 as the php version.
*/
pipeline {
    agent any
    environment {
        SSH_KEY_NAME        = 'jimbuntu'
        DEVELOP_HOST        = '10.0.54.186'
        STAGING_HOST        = '10.0.53.88'
        PRODUCTION_HOST     = '10.0.49.120'
    }
    stages {
        stage('Develop Feature'){
           when {
               not {
                  anyOf {
                    branch 'master';
                    branch 'develop'
                  }
               }
           }
            // step definitions
            steps {
                lintFeatureBranch()
            }
        }

        stage('Deploy Staging') {
            when {
                branch 'develop'
            }
            steps {
                deployStaging()
                echo 'We can deploy to staging'

                step([
                    $class:'CloverPublisher',
                    cloverReportDir: "${workspace}/reports/coverage",
                    cloverReportFileName: 'coverage.xml',
                    healthyTarget: [methodCoverage: 70, conditionalCoverage: 70, statementCoverage: 70],
                    unhealthyTarget: [methodCoverage: 50, conditionalCoverage: 50, statementCoverage: 50],
                    failingTarget: [methodCoverage: 0, conditionalCoverage: 0, statementCoverage: 0]
                ])
                step([
                $class: 'XUnitPublisher',
                tools: [
                    PHPUnit([
                        pattern: '**/reports/unitreport.xml',
                        skipNoTestFiles: true,
                        failIfNotNew: false,
                        deleteOutputFiles: false,
                        stopProcessingIfError: true
                        ])
                     ]
                ])
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: "${workspace}/reports/coverage/", reportFiles: 'html/index.html', reportName: 'Coverage Report', reportTitles: ''])

            }

        }
    }
}

def lintFeatureBranch() {
    sshagent(credentials : [env.SSH_KEY_NAME]) {

        def statusCode = sh(returnStatus: true, script: """ssh -oStrictHostKeyChecking=no -t ubuntu@\${DEVELOP_HOST} /bin/bash <<EOF

        sudo mkdir -p /var/www/spout/${env.BUILD_NUMBER}
        sudo chown ubuntu: /var/www/spout/${env.BUILD_NUMBER}
        cd /var/www/spout/${env.BUILD_NUMBER}
        git clone https://github.com/dextacy10-13/spout.git .
        git checkout ${env.BRANCH_NAME}
        git pull
        composer install
        vendor/bin/php-cs-fixer fix --config=.php_cs.dist -v --dry-run --stop-on-violation --using-cache=no
EOF
        """)
        if (statusCode == 0) {
          echo 'No issues'
          mergeBranchDevelop()
        } else {
          echo 'We have issues'
        }
       // sh 'scp ./source/filename user@hostname.com:/remotehost/target'
    }
}

def mergeBranchDevelop() {
    sshagent(credentials : [env.SSH_KEY_NAME]) {
    sh(returnStatus: true, script:"""ssh -oStrictHostKeyChecking=no -t ubuntu@\${DEVELOP_HOST} /bin/bash <<EOF
    cd /var/www/spout/${env.BUILD_NUMBER}
    git config --global user.email 'jamesmyersone@hotmail.com'
    git config --global user.name 'Mr James Myers'
    export GITHUB_TOKEN=${MY_GIT_TOKEN}
    git remote rm origin
    git remote add origin "https://dextacy10-13:${MY_GIT_TOKEN}@github.com/dextacy10-13/spout.git"
    git config --global url."https://api:${MY_GIT_TOKEN}@github.com/".insteadOf "https://github.com/"
    git config --global url."https://ssh:${MY_GIT_TOKEN}@github.com/".insteadOf "ssh://git@github.com/"
    git config --global url."https://git:${MY_GIT_TOKEN}@github.com/".insteadOf "git@github.com:"
    git checkout develop
    git merge ${env.BRANCH_NAME}

    git push origin develop
EOF
    """)
    }
}

def deployStaging() {
    sshagent(credentials : [env.SSH_KEY_NAME]) {

        def statusCode = sh(returnStatus: true, script:"""ssh -oStrictHostKeyChecking=no -t ubuntu@\${STAGING_HOST} /bin/bash <<EOF
        sudo mkdir -p /var/www/spout/${env.BUILD_NUMBER}
        sudo chown ubuntu: /var/www/spout/${env.BUILD_NUMBER}
        cd /var/www/spout/${env.BUILD_NUMBER}
        git clone https://github.com/dextacy10-13/spout.git .
        git checkout -b develop
        git pull
        composer install
        ./vendor/bin/phpunit --coverage-clover './reports/coverage/coverage.xml' --log-junit './reports/unitreport.xml' --coverage-html './reports/coverage/html' tests
EOF
        """)
        sh("sudo mkdir -p /home/ubuntu/build/reports/coverage")
        sh("sudo chown -R jenkins /home/ubuntu/build/reports")
        sh("""scp -o StrictHostKeyChecking=no -r ubuntu@${STAGING_HOST}:/var/www/spout/${env.BUILD_NUMBER}/reports/  ${workspace}/""")
    }
}
