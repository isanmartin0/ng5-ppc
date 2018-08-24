#!/usr/bin/groovy
import com.evobanco.AngularUtils
import com.evobanco.AngularConstants

def runAngularPPCJenkinsfile() {

    def utils = new AngularUtils()

    def angularNPMRepositoryURL = 'https://digitalservices.evobanco.com/artifactory/api/npm/angular-npm-repo/'
    def angularNPMLocalRepositoryURL = 'https://digitalservices.evobanco.com/artifactory/api/npm/angular-npm-local/'
    def angularGenericLocalRepositoryURL = 'https://digitalservices.evobanco.com/artifactory/angular-generic-local/'
    def artifactoryURL = 'https://digitalservices.evobanco.com/artifactory/'
    def artifactoryRepository = ''

    def openshiftURL = 'https://openshift.grupoevo.corp:8443'
    def openshiftCredential = 'openshift'
    def registry = '172.20.253.34'
    def artifactoryCredential = 'artifactory-token'
    def artifactoryNPMAuthCredential = 'artifactory-npm-auth'
    def artifactoryNPMEmailAuthCredential = 'artifactory-npm-email-auth'
    def jenkinsNamespace = 'cicd'
    def params
    String envLabel
    String branchName
    String branchNameHY
    String branchType


    //Parallel project configuration (PPC) properties
    def branchPPC = 'master'
    String credentialsIdPPCDefault = '4b18ea85-c50b-40f4-9a81-e89e44e20178' //credentials of the parallel configuration project
    def credentialsIdPPC
    def relativeTargetDirPPC = '/tmp/configs/PPC/'
    def isPPCJenkinsFile = false
    def isPPCJenkinsYaml = false
    def isPPCOpenshiftTemplate = false
    def jenkinsFilePathPPC = relativeTargetDirPPC + 'Jenkinsfile'
    def jenkinsYamlPathPPC = relativeTargetDirPPC + 'Jenkins.yml'
    def openshiftNginxTemplatePathPPC = relativeTargetDirPPC + 'kube/nginx_template.yaml'
    def jenknsFilePipelinePPC


    //Generic project configuration properties
    def gitDefaultProjectConfigurationPath='https://github.com/isanmartin0/evo-cicd-angular-generic-configuration'
    def relativeTargetDirGenericPGC = '/tmp/configs/generic/'
    def branchGenericPGC = 'master'
    def credentialsIdGenericPGC = '4b18ea85-c50b-40f4-9a81-e89e44e20178' //credentials of the generic configuration project
    def jenkinsYamlGenericPath = relativeTargetDirGenericPGC + 'Jenkins.yml'


    def packageJSON
    String projectURL
    String packageName
    String packageVersion
    String packageTag
    String packageTarball
    String packageViewTarball
    boolean isScopedPackage = false
    String packageScope


    int maxOldBuildsToKeep = 0
    int daysOldBuildsToKeep = 0

    //Taurus parameters
    def taurus_test_base_path = 'taurus'
    def acceptance_test_path = '/acceptance_test/'
    def performance_test_path = '/performance_test/'
    def smoke_test_path = '/smoke_test/'
    def security_test_path = '/security_test/'


    def openshift_route_hostname = ''
    def openshift_route_hostname_with_protocol = ''

    //Parameters Angular
    Boolean installGloballyAngularCli = false
    int image_stream_nodejs_version_default = 10
    def angularCliVersion_default = "6.1.2"
    def buildProdFlags_default = "--build-optimizer"
    def angularCliLocalParh = "node_modules/@angular/cli/bin/"

    def build_from_registry_url = 'https://github.com/isanmartin0/s2i-angular-container.git'
    def build_from_artifact_branch = 'master'

    def nodeJS_6_installation = "Node-6.11.3"
    def nodeJS_8_installation = "Node-8.9.4"
    def nodeJS_10_installation = "Node-10.8.0"

    def nodeJS_6_installation_angularCliVersion_default = "1.0.0"
    def nodeJS_8_installation_angularCliVersion_default = "1.7.4"
    def nodeJS_10_installation_angularCliVersion_default = "6.1.2"

    def nodeJS_6_installation_build_prod_flags = "--aot=false"
    def nodeJS_8_installation_build_prod_flags = "--aot=false"
    def nodeJS_10_installation_build_prod_flags = "--build-optimizer"

    def nodeJS_pipeline_installation = ""
    int image_stream_nodejs_version = image_stream_nodejs_version_default
    def angularCliVersion = angularCliVersion_default
    def buildProdFlags = buildProdFlags_default

    def packageJSONFilesNodeDistributionFolder = ["dist/"]


    echo "BEGIN ANGULAR GENERIC CONFIGURATION PROJECT (PGC)"

    node('nodejs10-chrome') {

        echo 'Pipeline begin timestamp... '
        sh 'date'

        stage('Checkout') {
            echo 'Getting source code...'
            checkout scm
            projectURL = scm.userRemoteConfigs[0].url
            echo "Source code hosted in: ${projectURL}"
        }


        try {
            def credentialsIdPPCArray = scm.userRemoteConfigs.credentialsId
            credentialsIdPPC = credentialsIdPPCArray.first()
            echo "Using credentialsIdPPCDefault value for access to Parallel Project Configuration (PPC)"

        } catch (exc) {
            echo 'There is an error on retrieving credentialsId of multibranch configuration'
            def exc_message = exc.message
            echo "${exc_message}"

            credentialsIdPPC = credentialsIdPPCDefault
        }

        echo "credentialsIdPPC: ${credentialsIdPPC}"

        stage('Detect Angular Parallel project configuration (PPC)') {

            packageJSON = readJSON file: 'package.json'

            isScopedPackage = utils.isScopedPackage(packageJSON.name)
            echo "isScopedPackage: ${isScopedPackage}"

            if (isScopedPackage) {
                packageScope = utils.getPackageScope(packageJSON.name)
                echo "packageScope: ${packageScope}"
            }

            packageName = utils.getProject(packageJSON.name)
            echo "packageName: ${packageName}"
            packageVersion = packageJSON.version
            echo "packageVersion: ${packageVersion}"
            packageTag = utils.getPackageTag(packageJSON.name, packageVersion)
            echo "packageTag: ${packageTag}"
            packageTarball = utils.getPackageTarball(packageJSON.name, packageVersion)
            echo "packageTarball: ${packageTarball}"
            packageViewTarball = utils.getPackageViewTarball(packageJSON.name, packageVersion)
            echo "packageViewTarball: ${packageViewTarball}"


            try {
                def parallelConfigurationProject = utils.getParallelConfigurationProjectURL(projectURL)

                echo "Angular parallel configuration project ${parallelConfigurationProject} searching"

                retry (3)
                        {
                            checkout([$class                           : 'GitSCM',
                                      branches                         : [[name: branchPPC]],
                                      doGenerateSubmoduleConfigurations: false,
                                      extensions                       : [[$class           : 'RelativeTargetDirectory',
                                                                           relativeTargetDir: relativeTargetDirPPC]],
                                      submoduleCfg                     : [],
                                      userRemoteConfigs                : [[credentialsId: credentialsIdPPC,
                                                                           url          : parallelConfigurationProject]]])
                        }

                echo "Angular Parallel configuration project ${parallelConfigurationProject} exits"

                // Jenkinsfile
                isPPCJenkinsFile = fileExists jenkinsFilePathPPC

                if (isPPCJenkinsFile) {
                    echo "Angular Parallel configuration project Jenkinsfile... FOUND"
                } else {
                    echo "Angular Parallel configuration project Jenkinsfile... NOT FOUND"
                }


                // Jenkins.yml
                isPPCJenkinsYaml = fileExists jenkinsYamlPathPPC

                if (isPPCJenkinsYaml) {
                    echo "Angular Parallel configuration project Jenkins.yml... FOUND"
                } else {
                    echo "Angular Parallel configuration project Jenkins.yml... NOT FOUND"
                }

                // Openshift template (template.yaml)


                if (isPPCJenkinsYaml) {
                    //Take parameters of the parallel project configuration (PPC)
                    params = readYaml  file: jenkinsYamlPathPPC
                    echo "Using Jenkins.yml from Angular parallel project configuration (PPC)"

                    //The template is provided by parallel project configuration (PPC)
                    params.openshift.templatePath = relativeTargetDirPPC + params.openshift.templatePath

                    echo "params.openshift.templatePath: ${params.openshift.templatePath}"

                    isPPCOpenshiftTemplate = fileExists params.openshift.templatePath
                } else {
                    isPPCOpenshiftTemplate = fileExists openshiftNginxTemplatePathPPC
                }


                if (isPPCOpenshiftTemplate) {
                    echo "Angular Parallel configuration project Openshift template... FOUND"
                } else {
                    echo "Angular Parallel configuration project Openshift template... NOT FOUND"
                }


                echo "isPPCJenkinsFile : ${isPPCJenkinsFile}"
                echo "isPPCJenkinsYaml : ${isPPCJenkinsYaml}"
                echo "isPPCOpenshiftTemplate : ${isPPCOpenshiftTemplate}"

            }
            catch (exc) {
                echo 'There is an error on retrieving Angular parallel project configuration'
                def exc_message = exc.message
                echo "${exc_message}"
            }
        }

        stage('Load pipeline configuration') {


            if (!isPPCJenkinsFile || !isPPCJenkinsYaml || !isPPCOpenshiftTemplate) {
                currentBuild.result = AngularConstants.FAILURE_BUILD_RESULT
                throw new hudson.AbortException('The parallel project configuration has not mandatory elements')
            }

            //Take parameters of the parallel project configuration (PPC)
            params = readYaml  file: jenkinsYamlPathPPC
            echo "Using Jenkins.yml from parallel project configuration (PPC)"

            //The template is provided by parallel project configuration (PPC)
            params.openshift.templatePath = relativeTargetDirPPC + params.openshift.templatePath
            echo "Template provided by parallel project configuration (PPC)"

            assert params.openshift.templatePath?.trim()

            echo "params.openshift.templatePath: ${params.openshift.templatePath}"

        }

        stage('NodeJS initialization') {
            echo 'Node initializing...'

            /*************************************************************
             ************* IMAGE STREAM TAG NODE VERSION *****************
             *************************************************************/
            echo "params.imageStreamNodejsVersion: ${params.imageStreamNodejsVersion}"
            String imageStreamNodejsVersionParam = params.imageStreamNodejsVersion
            if (imageStreamNodejsVersionParam != null && imageStreamNodejsVersionParam.isInteger()) {
                image_stream_nodejs_version = imageStreamNodejsVersionParam as Integer
            }

            if (image_stream_nodejs_version >= 10) {
                echo "Assigning NodeJS installation ${nodeJS_10_installation}"
                nodeJS_pipeline_installation = nodeJS_10_installation
                echo "Assigning @angular/cli version ${nodeJS_10_installation_angularCliVersion_default}"
                angularCliVersion = nodeJS_10_installation_angularCliVersion_default
                buildProdFlags = nodeJS_10_installation_build_prod_flags
            } else if (image_stream_nodejs_version >= 8) {
                echo "Assigning NodeJS installation ${nodeJS_8_installation}"
                nodeJS_pipeline_installation = nodeJS_8_installation
                echo "Assigning @angular/cli version ${nodeJS_8_installation_angularCliVersion_default}"
                angularCliVersion = nodeJS_8_installation_angularCliVersion_default
                buildProdFlags = nodeJS_8_installation_build_prod_flags
            } else if (image_stream_nodejs_version >= 6) {
                echo "Assigning NodeJS installation ${nodeJS_6_installation}"
                nodeJS_pipeline_installation = nodeJS_6_installation
                echo "Assigning @angular/cli version ${nodeJS_6_installation_angularCliVersion_default}"
                angularCliVersion = nodeJS_6_installation_angularCliVersion_default
                buildProdFlags = nodeJS_6_installation_build_prod_flags
            } else {
                currentBuild.result = "FAILED"
                throw new hudson.AbortException("Error setting NodeJS version")
            }

            def node = tool name: "${nodeJS_pipeline_installation}", type: 'jenkins.plugins.nodejs.tools.NodeJSInstallation'
            env.PATH = "${node}/bin:${env.PATH}"

            echo 'Node version:'
            sh "node -v"

            echo 'NPM version:'
            sh "npm -v"

        }

        stage('Prepare') {
            echo "Prepare stage (PGC)"

            angularSetDisplayName()

            echo "${currentBuild.displayName}"

            branchName = utils.getBranch()
            echo "We are on branch ${branchName}"
            branchType = utils.getBranchType(branchName)
            echo "This branch is a ${branchType} branch"
            branchNameHY = branchName.replace("/", "-").replace(".", "-").replace("_","-")
            echo "Branch name processed: ${branchName}"

        }

        stage ('Openshift environment') {
            switch (branchType) {
                case 'feature':
                    echo "Detect feature type branch"
                    envLabel="dev"
                    break
                case 'develop':
                    echo "Detect develop type branch"
                    envLabel="dev"
                    break
                case 'release':
                    echo "Detect release type branch"
                    envLabel="uat"
                    break
                case 'master':
                    echo "Detect master type branch"
                    envLabel="pro"
                    break
                case 'hotfix':
                    echo "Detect hotfix type branch"
                    envLabel="uat"
                    break
            }
            echo "Environment selected: ${envLabel}"
        }

        withCredentials([string(credentialsId: "${artifactoryNPMAuthCredential}", variable: 'ARTIFACTORY_NPM_AUTH'), string(credentialsId: "${artifactoryNPMEmailAuthCredential}", variable: 'ARTIFACTORY_NPM_EMAIL_AUTH')]) {
            withEnv(["NPM_AUTH=${ARTIFACTORY_NPM_AUTH}", "NPM_AUTH_EMAIL=${ARTIFACTORY_NPM_EMAIL_AUTH}"]) {
                withNPM(npmrcConfig: 'my-custom-npmrc') {

                    if (branchName != 'master') {


                        echo "params.angularCli.installGloballyAngularCli: ${params.angularCli.installGloballyAngularCli}"

                        if (params.angularCli.installGloballyAngularCli) {
                            installGloballyAngularCli = params.angularCli.installGloballyAngularCli.toBoolean()
                        }

                        if (installGloballyAngularCli) {

                            stage('Install globally @angular/cli') {

                                Boolean installAngularCliSpecificVersion = false
                                echo "params.angularCli.installAngularCliSpecificVersion: ${params.angularCli.installAngularCliSpecificVersion}"

                                if (params.angularCli.installAngularCliSpecificVersion) {
                                    installAngularCliSpecificVersion = params.angularCli.installAngularCliSpecificVersion.toBoolean()
                                }

                                if (installAngularCliSpecificVersion) {

                                    echo "Installing a specific @angular/cli version"
                                    echo "params.angularCli.angularCliVersion: ${params.angularCli.angularCliVersion}"
                                    String angularCliVersionParam = params.angularCli.angularCliVersion

                                    if (angularCliVersionParam != null) {
                                        angularCliVersion = angularCliVersionParam
                                    }

                                } else {
                                    echo "Installing default @angular/cli version"
                                    echo "NodeJS version: ${nodeJS_pipeline_installation}"
                                    echo "@angular/cli default version: ${angularCliVersion}"
                                }

                                echo "Installing globally @angular/cli version ${angularCliVersion}"
                                sh "npm install -g @angular/cli@${angularCliVersion}"

                            }
                        } else {
                            echo "Skipping @angular/cli installation..."
                        }


                        stage('Configure Artifactory NPM Registry') {
                            echo 'Setting Artifactory NPM registry'
                            sh "npm config set registry ${angularNPMRepositoryURL} "

                            sh "npm config get registry"
                        }

                        stage('Build') {
                            echo 'Building dependencies...'
                            sh 'npm i'
                        }



                        stage('Get ng version') {
                            echo 'ng version::'

                            if (installGloballyAngularCli) {
                                sh "ng version"
                            } else {
                                sh "${angularCliLocalParh}ng version"
                            }

                        }


                        if (branchType in params.testing.predeploy.unitTesting) {
                            stage('Test') {

                                echo 'TODO: Executing unit tests....'

                            }
                        } else {
                            echo "Skipping unit tests..."
                        }


                        if (branchType in params.testing.predeploy.sonarQube) {

                            stage('SonarQube') {

                                echo "TODO: Running SonarQube..."

                            }

                        } else {
                            echo "Skipping Running SonarQube..."
                        }

                        stage('Build Angular application') {

                            echo "Building angular application"

                            /***********************************************************
                             ************* BUILD PRODUCTION PARAMETERS *****************
                             ***********************************************************/

                            Boolean useBuildProdFlags = false
                            echo "params.ngBuildProd.useBuildProdFlags: ${params.ngBuildProd.useBuildProdFlags}"
                            echo "params.ngBuildProd.buildProdFlags: ${params.ngBuildProd.buildProdFlags}"

                            if (params.ngBuildProd.useBuildProdFlags) {
                                useBuildProdFlags = params.ngBuildProd.useBuildProdFlags.toBoolean()
                            }

                            if (useBuildProdFlags) {
                                buildProdFlags = params.ngBuildProd.buildProdFlags
                            } else {
                                echo "Build prod parameters default: ${buildProdFlags}"
                            }

                            echo "useBuildProdFlags: ${useBuildProdFlags}"
                            echo "buildProdFlags: ${buildProdFlags}"

                            if (installGloballyAngularCli) {
                                sh "ng build --prod ${buildProdFlags}"
                            } else {
                                sh "${angularCliLocalParh}ng build --prod ${buildProdFlags}"
                            }


                        }


                        stage('Create tarball') {

                            echo "Original package.json:"
                            echo "${packageJSON}"

                            def packageJSONFilesNode = packageJSON.files
                            echo "packageJSONFilesNode: ${packageJSONFilesNode}"

                            //Redefining packageJSON.files
                            packageJSON.files = packageJSONFilesNodeDistributionFolder

                            def packageJSONPrivateNode = packageJSON.private
                            echo "packageJSONPrivateNode: ${packageJSONPrivateNode}"

                            //Redefining packageJSON.private
                            if (packageJSONPrivateNode) {
                                packageJSON.private = false
                            }

                            echo "Updated package.json:"
                            echo "${packageJSON}"

                            writeJSON file: 'package.json', json: packageJSON, pretty: 4

                            sh "npm pack"
                        }

                        stage ('Check tarball creation') {

                            try {
                                echo 'Check tarball creation ...'
                                tarball_creation_script = $/eval "ls ${packageTarball}"/$
                                echo "${tarball_creation_script}"
                                def tarball_creation_view = sh(script: "${tarball_creation_script}", returnStdout: true).toString().trim()
                                echo "${tarball_creation_view}"
                            } catch (exc) {
                                echo 'There is an error on tarball creation'
                                def exc_message = exc.message
                                echo "${exc_message}"
                                currentBuild.result = "FAILED"
                                throw new hudson.AbortException("Error checking existence of tarball")
                            }
                        }

                    } else {

                        stage('Configure Artifactory NPM Registry') {
                            echo 'Setting Artifactory NPM registry'
                            sh "npm config set registry ${angularNPMRepositoryURL} "

                            sh "npm config get registry"
                        }
                    }


                    stage('Setting S2I project') {

                        echo "Setting source code to build from URL (build from registry package)"
                        echo "Source URL: ${projectURL} --> ${build_from_registry_url}"
                        projectURL = build_from_registry_url
                        echo "new projectURL: ${projectURL}"
                        echo "Setting source code to build from branch (build from registry package)"
                        echo "Source branch: ${branchName} --> ${build_from_artifact_branch}"
                        branchName = build_from_artifact_branch
                        echo "new branchName: ${branchName}"

                    }
                }
            }
        }


    } // end of node

} //end of method

return this

