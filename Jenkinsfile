def functionName = 'MoviesLoader'
def imageName = 'adriannavarro/loader'
def bucket = 'deployment-packages-watchlist.test123456'
def region = 'eu-west-1'

node('jenkins_agent'){
    try {
        stage('Checkout'){
            checkout scm
        }

        stage('Unit Tests'){
            def imageTest= docker.build("${imageName}-test", "-f Dockerfile.test .")
            imageTest.inside{
                sh "python test_index.py"
            }
        }

        stage('Build'){
            sh "zip -r ${commitID()}.zip index.py movies.json"
        }

        stage('Push'){
            sh "aws s3 cp ${commitID()}.zip s3://${bucket}/${functionName}/"
        }

        stage('Deploy'){
            sh "aws lambda update-function-code --function-name ${functionName} \
                    --s3-bucket ${bucket} --s3-key ${functionName}/${commitID()}.zip \
                    --region ${region}"

            sh "aws lambda publish-version --function-name ${functionName} \
                    --description ${commitID()} --region ${region}"
        }
    } catch(e){
        currentBuild.result = 'FAILED'
        throw e
    } finally {
        sh "rm ${commitID()}.zip"
    }
}

def commitAuthor(){
    sh 'git show -s --pretty=%an > .git/commitAuthor'
    def commitAuthor = readFile('.git/commitAuthor').trim()
    sh 'rm .git/commitAuthor'
    commitAuthor
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}

def commitMessage() {
    sh 'git log --format=%B -n 1 HEAD > .git/commitMessage'
    def commitMessage = readFile('.git/commitMessage').trim()
    sh 'rm .git/commitMessage'
    commitMessage
}