pipeline {
    agent any

    environment {
        // 1. 안드로이드 SDK 경로 (본인 PC 경로)
        ANDROID_HOME = "/Users/jayden.coys/Library/Android/sdk"
        
        // 2. PATH에 platform-tools 추가 (adb 명령어 사용을 위해 필수)
        PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/cmdline-tools/latest/bin:${PATH}"
        
        // 3. GitHub에서 코드를 받아올 작업 공간 설정
        BASE_DIR = "${env.WORKSPACE}" 
    }

    stages {
        stage('Checkout') {
            steps {
                echo "GitHub에서 최신 코드를 가져옵니다..."
                // SCM 설정에 따라 자동으로 체크아웃됩니다.
            }
        }

        stage('Check ADB') {
            steps {
                script {
                    echo "ADB 연결 상태를 확인합니다..."
                    sh "adb version"
                    sh "adb devices"
                }
            }
        }

        stage('Appium Server Start') {
            steps {
                script {
                    echo "Appium 서버를 시작합니다..."
                    sh "pkill -f appium || true"
                    // 로그 파일 경로도 Workspace 기준으로
                    sh 'nohup appium -p 4723 --allow-insecure=adb_shell > appium_4723.log 2>&1 &'
                    sh 'nohup appium -p 4725 --allow-insecure=adb_shell > appium_4725.log 2>&1 &'
                    sleep 10
                }
            }
        }

        stage('Execute Daum Search Test') {
            steps {
                sh '''
                cd ${BASE_DIR}
                
                # 가상환경 설정
                if [ ! -d "venv" ]; then
                    python3 -m venv venv
                fi
                . venv/bin/activate
                
                # 라이브러리 설치
                pip install --upgrade pip
                pip install Appium-Python-Client pytest gspread oauth2client google-generativeai Pillow requests google-genai
                
                # 기존 결과 삭제
                rm -f results.xml
                
                # --- [수정된 부분] ---
                echo "테스트를 시작합니다..."
                
                # tests 폴더 안에 있는 test_*.py 파일들을 모두 실행
                # 또는 특정 파일만 실행하려면: 
                pytest -v -s --junitxml=results.xml tests/daum_v1_test.py || true
                '''
            }
        }
    }

    post {
        always {
            echo "테스트 종료. 결과를 정리합니다."
            junit testResults: "results.xml", allowEmptyResults: true
            sh "pkill -f appium || true"
            archiveArtifacts artifacts: "*.xml, **/*.log, **/*.png", allowEmptyArchive: true
        }
    }
}
