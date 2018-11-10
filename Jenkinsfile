def isReleaseVersion(){
    return env.BRANCH_NAME == 'master'
}

def formatVersion(def version) {
    if (!isReleaseVersion()) {
        return "${version}-${env.BRANCH_NAME}${env.BUILD_ID}"
    }
    return "${version}"
}

def productionImageName = "renehr9102/bitknown_ghost"
def testImageName = "bitknown_test"

timestamps {
    node('master') {
        checkout scm
        def version = ''

        try{
            def testDockerfile = 'Dockerfile.test'
            stage('Build Test Image') {
                sh "docker build -f ${testDockerfile} -t $testImageName ./"
            }

            stage('Run Tests') {
                sh "docker run --rm $testImageName yarn test"
            }

            stage('Get Version') {
                def testImage = docker.image(testImageName) 
                
                testImage.inside {
                    def packageJSON = readJSON file: 'package.json'
                    version = packageJSON.version
                }
            }

            stage('Build Production Image') {
                def formattedVersion = formatVersion(version)
                def image = docker.build("$productionImageName:$formattedVersion")

                stage("Publish image with tag $formattedVersion") {
                    docker.withRegistry('', 'dockerhub') {
                        image.push()

                        if (isReleaseVersion()) {
                            stage('Update latest tag') {
                                image.push('latest')
                            }
                        }
                    }
                }
            }
        }
        finally {
            stage("Cleanup images") {
                sh "docker rmi $testImageName:latest"

                if (version) {
                    sh "docker rmi $productionImageName:${formatVersion(version)}"
                    sh "docker rmi $productionImageName:latest"
                }
            }
        }
    }
}