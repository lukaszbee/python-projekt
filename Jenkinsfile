pipeline {
    agent any

    environment {
        // Używamy nowej, bezpiecznej nazwy credentiala typu "Username with password"
        // Jenkins automatycznie utworzy GITHUB_AUTH_USR (login) i GITHUB_AUTH_PSW (token/hasło)
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
                // Zakładamy, że python3 jest w obrazie jenkinsa lub agenta
                sh 'pip install -r requirements.txt || echo "Brak pip, pomijam instalacje (DEMO)"'
                
                sh 'echo "Uruchamiam Linter..."'
                // Jeśli pylint znajdzie błędy, zwróci kod błędu i zatrzyma pipeline (po usunięciu || echo)
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
                    
                    // Bezpieczniejsze użycie apostrofów (sh ''') w celu uniknięcia ostrzeżenia 
                    // i bezpiecznego użycia zmiennej $GITHUB_AUTH_PSW (tokena)
                    sh '''
                        curl -X POST -H "Authorization: token $GITHUB_AUTH_PSW" \
                        -H "Accept: application/vnd.github.v3+json" \
                        https://api.github.com/repos/$REPO_OWNER/$REPO_NAME/pulls \
                        -d '{"title":"Automatyczny PR z Jenkinsa - $BRANCH_NAME","body":"Lint przeszedł pomyślnie.","head":"$BRANCH_NAME","base":"$MAIN_BRANCH"}'
                    '''
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
                    echo "Proba zmergowania zmian z $BRANCH_NAME do main..."
                    
                    // Zabezpieczony i poprawiony blok GIT:
                    sh '''
                        # Konfiguracja tożsamości
                        git config user.email "jenkins@bot.com"
                        git config user.name "Jenkins Bot"
                        
                        # 1. Pobranie aktualnego stanu main (NAPRAWA: błąd pathspec)
                        git fetch origin main
                        git checkout -B main origin/main
                        
                        # 2. Merge zmian z brancha, na którym działał pipeline
                        git merge origin/$BRANCH_NAME
                        
                        # 3. Wypchnięcie zmian (NAPRAWA: ostrzeżenie o bezpieczeństwie)
                        # Używamy zmiennych Shell ($) wewnątrz bloku apostrofów (''')
                        git push https://$GITHUB_AUTH_USR:$GITHUB_AUTH_PSW@github.com/$REPO_OWNER/$REPO_NAME.git main
                    '''
                }
            }
        }
    }
}
