import hudson.model.*

m1 = System.currentTimeMillis()
m2 = 0
durTime = 0

lock(resource: "${env.JOB_NAME}_${env.BRANCH_NAME}", inversePrecedence: false) {
    node() {
        deleteDir()
        step([$class: 'StashNotifier'])

        //################################
        // STATIC VAR!!!

        env.stepsMapNameJenkins = 'stepsMapNpm'
        env.projectName = "NPMMOD"

        // ###############################

        try {
            // System Env //
            env.serviceName = "${env.JOB_NAME}".split('/')[1]

            stage('Checkout') {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '34cd7bb4-0b2a-4dd6-8ef0-8e03b619cf1c', name: 'origin', refspec: '+refs/heads/master:refs/remotes/origin/master', url: 'https://bb.flowfact.com/scm/' + projectName + '/' + serviceName + '.git']]])
                sh "mkdir pipeline"
                dir('pipeline') {
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '34cd7bb4-0b2a-4dd6-8ef0-8e03b619cf1c', name: 'origin', refspec: '+refs/heads/master:refs/remotes/origin/master', url: 'https://bb.flowfact.com/scm/IN/jenkins-pipeline-steps.git']]])
                }
                sh "mkdir tests"
                dir('tests') {
                    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '34cd7bb4-0b2a-4dd6-8ef0-8e03b619cf1c', name: 'origin', refspec: '+refs/heads/master:refs/remotes/origin/master', url: 'https://bb.flowfact.com/scm/TT/jenkins-test-automation.git']]])
                }
            }

            func = load('pipeline/functions/func.groovy');
            def vars = load('pipeline/functions/loadVarsLibs.groovy');
            vars.loadVars()

            // Pipeline Steps - while
            count = 1
           	int stepsCount = stepsCount.toInteger()
            while (count <= stepsCount) {
                try {
                    name = "${count}_name"
                    env.step_name = mapping[name]
                    echo "step_name: "+step_name
                    stage("${step_name}") {
                        def steps = load('pipeline/functions/steps.groovy');
                        steps.stepToLoad(count)

                    }
                    count++;
                }
                catch (err) {
                   println err
                   throw err
                }
            }

            m2 = System.currentTimeMillis()
            durTime = func.durationTime(m1, m2)

            mapping = readProperties file: pwd() + '/pipeline/'+productName+'/'+stepsMapName+'.properties'
            step_job = mapping['end_ok_job']
            sload = "pipeline/"+productName+"/${step_job}.groovy"
            def end = load(sload);
            end.endPositive()

            currentBuild.result = 'SUCCESS'
            step([$class: 'StashNotifier'])

        } catch (err) {

            func = load('pipeline/functions/func.groovy');
            def vars = load('pipeline/functions/loadVarsLibs.groovy');
            vars.loadVars()

            mapping = readProperties file: pwd() + '/pipeline/'+productName+'/'+stepsMapName+'.properties'
            step_job = mapping['end_err_job']
            sload = "pipeline/"+productName+"/${step_job}.groovy"
            def end = load(sload);
            end.endNegative()

            currentBuild.result = 'FAILED'
            step([$class: 'StashNotifier'])

            println err
            throw err
        }
    }
}