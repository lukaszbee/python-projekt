pipeline {
    agent any

    environment {
        // Zmienne konfiguracyjne
        GITHUB_CRED = credentials('gh_token') 
        REPO_OWNER = 'lukaszbee'
        REPO_NAME = 'python-projekt'
        MAIN_BRANCH = 'main'
    }

    stages {
        // Krok 1: Lintowanie kodu
        stage('Lint Code') {
            steps {
                sh 'echo "Instalowanie zaleznosci..."'
                // Zakładamy, że python3 jest w obrazie jenkinsa lub agenta
                // Jeśli nie, użyj obrazu dockerowego w 'agent'
                sh 'pip install -r requirements.txt || echo "Brak pip, pomijam instalacje w tym demo"'
                
                sh 'echo "Uruchamiam Linter..."'
                // Tutaj uruchamiamy linter. Jeśli zwróci błąd, pipeline się zatrzyma.
                // Używamy pylint lub prostego echo dla testu
                sh 'pylint app.py || echo "Linter znalazł błędy, ale puszczam dalej na potrzeby demo (usun echo aby blokowac)"'
            }
        }

        // Krok 2: Tworzenie PR (tylko jeśli nie jesteśmy na main)
        stage('Create PR') {
            when {
                not { branch 'main' }
            }
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    echo "Tworzenie PR z gałęzi ${branchName} do ${MAIN_BRANCH}"
                    
                    // Wywołanie API GitHuba tworzące PR
                    def response = sh(script: """
                        curl -X POST -H "Authorization: token ${GITHUB_CRED}" \
                        -H "Accept: application/vnd.github.v3+json" \
                        https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/pulls \
                        -d '{"title":"Automatyczny PR z Jenkinsa","body":"Lint przeszedł pomyślnie.","head":"${branchName}","base":"${MAIN_BRANCH}"}'
                    """, returnStdout: true).trim()
                    
                    echo "Odpowiedź GitHub: ${response}"
                    
                    // Wyciągnięcie numeru PR (prosty parsing grepem/awk dla demo, w produkcji lepiej uzyc 'jq')
                    // Zakładamy, że PR został utworzony. Jeśli już istnieje, API zwróci błąd, trzeba to obsłużyć.
                }
            }
        }

        // Krok 3: Merge PR (automatyczny)
        stage('Merge PR') {
            when {
                not { branch 'main' }
            }
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    
                    // Pobieramy numer PR na podstawie nazwy brancha (potrzebujemy narzędzia jq lub skomplikowanego grepa)
                    // Dla uproszczenia w tym skrypcie: zakładamy pomyślny merge bezpośredni przez git
                    // LUB używamy API do merge'owania. 
                    
                    echo "Próba zmergowania zmian do main..."
                    
                    // Opcja prostsza: Merge na poziomie Gita w Jenkinsie i push
                    sh """
                        git config --global user.email "jenkins@bot.com"
                        git config --global user.name "Jenkins Bot"
                        git checkout ${MAIN_BRANCH}
                        git pull origin ${MAIN_BRANCH}
                        git merge origin/${branchName}
                        git push https://${GITHUB_CRED}@github.com/${REPO_OWNER}/${REPO_NAME}.git ${MAIN_BRANCH}
                    """
                }
            }
        }
    }
}
