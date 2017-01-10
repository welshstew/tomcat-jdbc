node('maven') {

    def gitUsername = ''
    def gitPassword = ''

    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: '431cbc19-9e57-4011-920d-02304cafc84c', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD']]){
        gitUsername = "${env.GIT_USERNAME}"
        gitPassword = "${env.GIT_PASSWORD}"
    }

    //def mvnHome = tool 'M3'
    def newVersion = "1.1.${env.BUILD_NUMBER}"

    stage('Checkout') {
        git branch: 'master', url: 'https://github.com/welshstew/tomcat-jdbc.git'
    }

    stage('Set Version') {
        sh "mvn --settings /etc/m2/settings.xml -f pom.xml versions:set -DnewVersion=${newVersion}"
    }

    stage('Build') {
        sh "mvn --settings /etc/m2/settings.xml -f pom.xml clean install -DskipTests"
    }

    stage('Unit Test'){
        sh "mvn --settings /etc/m2/settings.xml -f pom.xml test"
    }


    stage('Deploy and Tag'){
        sh "mvn --settings /etc/m2/settings.xml org.apache.maven.plugins:maven-deploy-plugin:2.8.2:deploy-file -Durl=http://nexus-ci.cloudapps-f109.oslab.opentlc.com/content/repositories/releases/ \
                                                                                -DrepositoryId=nexus \
                                                                                -Dfile=target/tomcat-jdbc.war \
                                                                                -DpomFile=pom.xml \
                                                                                -Dfiles=target/classes/META-INF/fabric8/openshift.yml,target/classes/META-INF/fabric8/openshift.json \
                                                                                -Dclassifiers=openshift,openshift \
                                                                                -Dtypes=yml,json \
                                                                                -Dversion=${newVersion}"

        sh("git config --global user.email \"stuart.winchester@gmail.com\"")
        sh("git config --global user.name \"Stuart Winchester\"")
        sh("git tag -a ${newVersion} -m 'Jenkins CI Tag'")
        sh("git push https://${gitUsername}:${gitPassword}@github.com/welshstew/tomcat-jdbc.git --tags")
    }

}
