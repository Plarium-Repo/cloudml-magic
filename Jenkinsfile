@Library('pdp-automation-integration')
import com.plarium.pdp.automation.integration.common.JobContext

node (label : "web-master"){

    parameters {
        booleanParam(
                defaultValue: false,
                description: 'Deploy (Option working only for QA branch  ?',
                name: 'DEPLOY_PYCORE'
        )

        string(
                defaultValue: false,
                description: 'Deploy (Option working only for QA branch  ?',
                name: 'PYCORE_VERSION_TO_DEPLOY'
        )
        booleanParam(
                defaultValue: false,
                description: 'Build',
                name: 'SETUP_PYCORE_ENV'
        )
        booleanParam(
                defaultValue: false,
                description: 'Release (Warning: if set to true, this option should bump the SNAPSHOT version en develo branch)',
                name: 'RELEASE_PYCORE'
        )
        string(
                defaultValue: "int",
                description: 'Release (Warning: if set to true, this option should bump the SNAPSHOT version en develo branch)',
                name: 'ENV_TYPE'
        )
    }

    checkoutGitVersion(params)

    JobContext context = genPyCoreConfiguration(params)

    String project_path = """${env.WORKSPACE}"""

    def builds = [:]

    if(context.envType  == 'int') {
        builds["INT - ${context.projectName} Build/Publish/Deploy"] = {
            //deployPyCore(context)
            sshagent(credentials: ['16d4ee75-d9a7-4030-9b3b-28c3729fc651']) {
                sh "cd ${project_path}"
                sh """python setup.py sdist --formats=gztar"""
                sh "cd dist"
                //echo """${context.deliverableVersion}"""
                String version = context.deliverableVersion.replaceAll("[^a-zA-Z0-9.]","")
                //TODO : "cloudmlmagic" ---> generic magic name to get out from code
                echo """move "./cloudmlmagic-${version}.tar.gz"  to folder '/root/packages/'"""
                sh """sudo mv ${project_path}/dist/cloudmlmagic-${version}.tar.gz /root/packages/"""
            }
            releasePyCore(context)
        }
    }  else if(context.envType in ['qa', 'pre','prod','dev']) {
        builds["${context.envType.toUpperCase()} - ${context.projectName} Deploy"] = {
            echo "versionToDeploy : ${context.versionToDeploy}"
            if (context.versionToDeploy == null || context.versionToDeploy.size() <= 0) {
                error("No selected version ! Aborting Deployment")
            } else {
                echo "Selected version to deploy : ${context.versionToDeploy}"
                node(label: "web-master"){
                    checkoutGitVersion(params)
                    deployPyCore(context)
                }

            }
        }
    } else {
        builds["CORE - No valid GIT branch detected"] = {
            node {
                stage("Do Nothing"){
                    sh """echo "Branch :"  ${current_branch_name} " wont be built/deployed !!!" """
                }
            }
        }
    }

    parallel builds

}