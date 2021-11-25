pipeline
{
    agent
    {
        label 'slave'
    }

    tools
    {
        maven 'M3'
    }

    environment 
    {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }



    stages 
    {
        stage('Clear running apps')
        {
            steps
            {
                // Clear previous instaces of app built
                sh 'docker rm -f pandaapp || true'
            }
        }
        stage('Get Code')
        {
            steps
            {
                // Get Code
                git branch: 'selenium', url: 'https://github.com/RokDavid/panda_application.git'
            }
        }
        stage('Build and Junit')
        {
            steps
            {
                // Run Maven on a Unnix agent
                sh 'mvn clean install -DskipTests=true'
            }
        }
        stage('Build Docker image')
        {
            steps
            {
                sh 'mvn package -Pdocker'
            }
        }
        stage('Run Docker app')
        {
            steps
            {
                sh 'docker run -d -p 0.0.0.0:8080:8080 --name pandaapp ${IMAGE}:${VERSION}'
            }
        }
        stage('Test Selenium')
        {
            steps
            {
                sh 'mvn test -Pselenium'
            }
        }
        stage('Deploy jar to artifactory')
        {
            steps
            {
                configFileProvider([configFile(fileId: 'e200bda1-2d6b-493c-a082-496e22904159', variable: 'mvnset')])
                {
                    sh "mvn -s $mvnset deploy -Dmaven.test.skip=ture -e"
                }
            }
        }
    }
    post
    {
        always
        {
            sh 'docker stop pandaapp'
            deleteDir()
        }
    }
}