stage 'Compile'
node ('docker-cloud') {
    checkout scm
    mvn 'clean package'
    dir('target') {stash name: 'jar', includes: 'x.jar'}
}

stage 'QA'
parallel(longerTests: {
    runTests(30)
}, quickerTests: {
    runTests(20)
})

stage name: 'Staging', concurrency: 1
node ('docker-cloud') {
    deploy 'staging'
}

input message: "Does staging look goosd?"
try {
    checkpoint('Before production')
} catch (NoSuchMethodError _) {
    echo 'Checkpoint feature available in CloudBees Jenkins Enterprise.'
}

stage name: 'Production', concurrency: 1
node ('docker-cloud'){
    echo 'Production server looks to be alive'
    deploy 'production'
    echo "Deployed to production"
}

def mvn(args) {
    sh "${tool 'Maven 3.x'}/bin/mvn ${args}"
}

def runTests(duration) {
    node {
        sh "sleep ${duration}"
        }
    }

def deploy(id) {
    unstash 'jar'
    sh "cp x.jar /tmp/${id}.jar"
}

def undeploy(id) {
    sh "rm /tmp/${id}.jar"
}
