
node( 'built-in' ) {

  Boolean isPR = env?.GITHUB_PR_TARGET_BRANCH ?: false
  try {
    cleanWs()
    checkout scmGit(
               branches: [[ name: "origin-pull/$GITHUB_PR_NUMBER/$GITHUB_PR_COND_REF" ]],
               browser: github( 'https://github.com/marslojiao-mvl/webhook' ),
               extensions: [],
               userRemoteConfigs: [[
                 credentialsId: 'GITHUB_SSH_CREDENTAIL',
                 name: 'origin-pull',
                 refspec: "+refs/pull/$GITHUB_PR_NUMBER/*:refs/remotes/origin-pull/$GITHUB_PR_NUMBER/*",
                 url: 'git@github.com:marslojiao-mvl/webhook.git'
               ]]
             )

    if ( isPR ) {
      gitHubPRStatus githubPRMessage( "${env.GITHUB_PR_COND_REF} run started" )
      println ">> build triggered by : pull request."
    } else {
      println '>> Build was not started by github pull request.'
    }

  } catch ( Exception e ) {
    if ( isPR ) { githubPRAddLabels labelProperty: labels( 'ERROR' ) }
  } finally {

    if ( isPR ) {
      String prLabel = ( 'SUCCESS' == currentBuild.currentResult ) ? 'VERIFIED' : 'FAILED'
      githubPRAddLabels labelProperty: labels( prLabel )
      githubPRComment comment: githubPRMessage( "${env.GITHUB_PR_HEAD_SHA} ${currentBuild.currentResult} in [${currentBuild.fullDisplayName}](${env.BUILD_URL})" )
      setGitHubPullRequestStatus context: "PR #${env.GITHUB_PR_NUMBER} ${currentBuild.currentResult} in ${currentBuild.fullDisplayName}",
                                 message: "CI build successfully in build ${currentBuild.fullDisplayName} ${BUILD_DISPLAY_NAME}",
                                 state: 'SUCCESS'

      step([ $class: 'GitHubCommitStatusSetter',
             errorHandlers: [[ $class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE" ]],
             statusBackrefSource: [ $class: "ManuallyEnteredBackrefSource", backref: "${BUILD_URL}console/" ],
             statusResultSource: [
               $class: 'ConditionalStatusResultSource',
               results: [
                          [
                            $class  : 'AnyBuildResult',
                            message : "${currentBuild.fullDisplayName} ${currentBuild.currentResult} !",
                            state   : "${currentBuild.currentResult}"
                          ]
               ]
             ]
      ])
    }

  } // try | catch | finally
} // node
