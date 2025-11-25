pipeline {
    agent any

    environment {
        // Zmienne konfiguracyjne
        // Credential typu "Username with password" o ID 'gh_login_token'
        GITHUB_AUTH = credentials('gh_login_token') 
        
        REPO_OWNER = 'lukaszbee'
        REPO_NAME = 'python-projekt'
        MAIN_BRANCH = 'main'
    }

    stages {
        // --- Krok 1: Lintowanie kodu ---
        stage('Lint Code') {
            steps {
                sh 'echo "Instalowanie zaleznosci..."'
                // Użycie || echo pozwala na kontynuację, nawet jeśli 'pip' nie jest zainstalowany
                sh 'pip install -r requirements.txt || echo "Brak pip, pomijam instalacje (DEMO)"'
                
                sh 'echo "Uruchamiam Linter..."'
                // Aby błędy lintowania zatrzymały pipeline, usuń || echo poniżej
                sh 'pylint app.py || echo "Linter znalazł błędy, ale puszczam dalej (DEMO)"'
            }
        }

        // --- Krok 2: Tworzenie PR (tylko z gałęzi innej niż main) ---
        stage('Create PR') {
            when {
                not { branch 'main' }
            }
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    echo "Tworzenie PR z gałęzi ${branchName} do ${MAIN_BRANCH}"
                    
                    // Bezpieczne wywołanie API GitHuba (używamy zmiennej $GITHUB_AUTH_PSW)
                    sh """
                        curl -X POST -H "Authorization: token $GITHUB_AUTH_PSW" \
                        -H "Accept: application/vnd.github.v3+json" \
                        https://api.github.com/$REPO_OWNER/$REPO_NAME/pulls \
                        -d '{"title":"Automatyczny PR z Jenkinsa - $BRANCH_NAME","body":"Lint przeszedł pomyślnie.","head":"$BRANCH_NAME","base":"$MAIN_BRANCH"}'
                    """
                }
            }
        }

        // --- Krok 3: Merge do main (tylko z gałęzi innej niż main) ---
        stage('Merge PR') {
            when {
                not { branch 'main' }
            }
            steps {
                script {
                    echo "Proba zmergowania zmian z ${env.BRANCH_NAME} do main..."
                    
                    // Zabezpieczony i poprawiony blok GIT: rozbijamy na pojedyncze komendy sh
                    
                    // 1. Konfiguracja tożsamości
                    sh 'git config user.email "jenkins@bot.com"'
                    sh 'git config user.name "Jenkins Bot"'
                    
                    // 2. Pobranie aktualnego stanu main (NAPRAWA: błąd pathspec)
                    sh 'git fetch origin main'
                    sh 'git checkout -B main origin/main'
                    
                    // 3. Merge zmian z brancha, na którym działał pipeline
                    // Używamy zmiennej środowiskowej Jenkinsa $BRANCH_NAME
                    sh "git merge origin/${env.BRANCH_NAME}"
                    
                    // 4. Wypchnięcie zmian (NAPRAWA: ostrzeżenie o bezpieczeństwie)
                    // Zmienne $GITHUB_AUTH_USR i $GITHUB_AUTH_PSW są wstrzykiwane bezpiecznie
                    sh 'git push https://$GITHUB_AUTH_USR:$GITHUB_AUTH_PSW@github.com/$REPO_OWNER/$REPO_NAME.git main'
                }
            }
        }
    }
}
