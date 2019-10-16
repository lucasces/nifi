def registry = "docker-snapshots.devops.somosdigital.io"
def registrySecrets = "docker-releases"
def version = "1.9.2"
def imageName = "apache/nifi:${version}"
def dockerOpts = '--network container:\$(docker ps | grep \$(hostname) | grep k8s_POD | cut -d\" \" -f1) .'

podTemplate(name: 'nifi-build',
        label: 'nifi-build',
        inheritFrom: 'base',
        containers: [
                containerTemplate(name: 'docker',
                        image: 'docker:stable',
                        ttyEnabled: true,
                        command: 'cat',
                        alwaysPullImage: true),

                containerTemplate(name: 'java-maven',
                        image: 'docker.devops.somosdigital.io/build-images/maven:latest',
                        ttyEnabled: true,
                        command: 'cat',
                        alwaysPullImage: true)
        ]) {
    node('nifi-build') {
        stage('Clone') {
            checkout scm
        }

        configFileProvider(
                [configFile(fileId: '9c7d49b5-d9b2-43d2-84eb-f67d49a9a175',
                        targetLocation: '/home/jenkins/.m2/settings.xml'),
                 configFile(fileId: '404d290e-74cc-4d14-9302-e034b1780606',
                         targetLocation: '/home/jenkins/.docker/config.json')]) {

            container('java-maven') {
                stage('Build assemblies') {
                    sh 'mvn -B -DskipTests clean install'
                }

                stage('Copy assemblies') {
                    sh "cp nifi-assembly/target/nifi-1.9.2-bin.zip nifi-docker/dockerhub/"
                    sh "cp nifi-toolkit/nifi-toolkit-assembly/target/nifi-toolkit-1.9.2-bin.zip nifi-docker/dockerhub/"
                }
            }
            container('docker') {
                stage('Build image') {
                    dir("nifi-docker/dockerhub") {
                        docker.withRegistry(registry, registrySecrets) {
                            def buildImage = docker.build(imageName, dockerOpts)
                            buildImage.push()
                            buildImage.push('latest')
                        }
                    }
                }
            }
        }
    }
}
