 /* node {  
   
    def mvnHome = tool 'maven-3.5.2'

    stage('Clone repo'){
        git branch: 'devops', url: 'https://github.com/SHODELACAILLE/Jenkins_Test.git'
    }

    stage('Build project'){
        dir('demo') { // Change le répertoire courant en 'demo' où se trouve le pom.xml.
            // Compile le projet sans exécuter les tests
            sh "'${mvnHome}/bin/mvn' -B -DskipTests clean package"
            
            // Génère le Javadoc
            sh "'${mvnHome}/bin/mvn' javadoc:javadoc"
        }
    }

    stage('Publish Javadoc') {
        // Assume que le Javadoc est généré dans 'target/site/apidocs' dans le répertoire 'demo'
        dir('demo/target/site/apidocs') {
            // Utilisez un plugin ou une méthode pour publier le Javadoc ici.
            // Par exemple, si vous utilisez le plugin HTML Publisher dans Jenkins:
            publishHTML(target: [
                reportDir: '.', // Le répertoire actuel contient les fichiers Javadoc
                reportFiles: 'index.html', // Le fichier d'entrée principal du Javadoc
                reportName: "Javadoc", // Le nom sous lequel le rapport doit être publié
                keepAll: true, // Conserve les rapports de toutes les builds
                allowMissing: false // Ne permet pas de signaler un build comme réussi si le dossier est manquant
            ])
        }
    }
}



pipeline {
    agent any 

    tools {
        // Spécifier l'installation de Maven
        maven 'maven-3.5.2'
    }

    stages {
        stage('Build project') {
            steps {
                // Naviguer vers le répertoire du projet
                dir('demo') {
                   sh 'chmod +x mvnw'

                    sh './mvnw -DskipTests  clean package'
                    sh './mvnw javadoc:javadoc'
                }
            }
        }
    }
*/

pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('SonarQube analysis') {
            steps {
                script {
                    // Assurez-vous que SONAR_HOST et SONAR_TOKEN sont définis
                    docker.run("--rm",
                        "-e SONAR_HOST_URL=${env.SONAR_HOST} " +
                        "-e SONAR_LOGIN=${env.SONAR_TOKEN} " +
                        "sonarsource/sonar-scanner-cli",
                        "sonar-scanner")
                }
            }
        }
    }


    post {
       
        success {
            script {
              
                if (fileExists('**/target/surefire-reports/*.xml')) {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
            
        }
       
    }
} 

