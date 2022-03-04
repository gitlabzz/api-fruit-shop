node {

    def mvnHome
    def branchName
    def targetEnvironment
    def pullRequest

    stage('Initialize') {
        branchName = BRANCH_NAME
        if (branchName.toUpperCase().startsWith("PR-")) {
            echo "found pull request '${branchName}', so targetting it to the 'DEV' environment!!!"
            echo "Checking out pull request  ========================================> ${branchName}"
            pullRequest = branchName.substring(branchName.lastIndexOf("-") + 1)
            echo "Pull request is   ========================================>  ${pullRequest}."
        }
    }

    stage("Checkout Code (${pullRequest ? 'PR-' + pullRequest : branchName})") {
        if (pullRequest) {
            try {
                git branch: '${BRANCH_NAME}', credentialsId: '2bc605b8-3d32-4c7b-84e2-4d858bc31c46', url: 'https://github.com/gitlabzz/demo-api.git'
            }
            catch (exception) {
                sh '''
                    git fetch origin +refs/pull/''' + pullRequest + '''/merge
                    git checkout FETCH_HEAD
                '''
                // as we know, branch name must be same as environment name in src/main/resources to pick config/env.properties file by apim-cli utility.
                branchName = "dev"
            }

            echo "Check out from pull request '${BRANCH_NAME}' is successfully completed!"
        } else {
            echo "Checking out branch  ========================================> ${BRANCH_NAME}"
            git branch: '${BRANCH_NAME}', credentialsId: '2bc605b8-3d32-4c7b-84e2-4d858bc31c46', url: 'https://github.com/gitlabzz/demo-api.git'
            echo "Check out from '${BRANCH_NAME}' is successfully completed!"
        }

    }

    stage("Prepare Environment (${branchName})") {
        targetEnvironment = branchName.toUpperCase()
        mvnHome = tool 'M3'

        echo "Target Environment is ---------------------------------> ${targetEnvironment}"

        withCredentials([usernamePassword(credentialsId: "APIM_ADMIN_USERNAME_PASSWORD_${targetEnvironment}_ENV", passwordVariable: 'password', usernameVariable: 'username')]) {
            env.APIM_ADMIN_USER = username
            env.APIM_ADMIN_PASSWORD = password
            env.AXWAY_APIM_CLI_HOME = "src/main/environments/${branchName}"

            echo "Using 'conf/env.properties' file from ---------------------------------> ${AXWAY_APIM_CLI_HOME}"
        }
    }

    stage("Publish API (${branchName})") {
        // Run the maven build
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                echo "Publishing to environment: '${branchName}'"
                env.MAVEN_OPTS = '-Xms256m -Xmx512m -Dlog4j.configurationFile=src/main/resources/log4j/log4j2.xml'
                sh '"$MVN_HOME/bin/mvn" exec:java'
            }
        }
    }

    stage('Prepare Package') {
        // Run the maven build
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                sh '"$MVN_HOME/bin/mvn" clean install'
            }
        }
    }

    stage('Push Package to Nexus') {
        // Run the maven build
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                // TODO: later make it deploy instead of install
                sh '"$MVN_HOME/bin/mvn" clean install'
            }
        }
    }
}