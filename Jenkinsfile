pipeline {
    agent { label 'Windows' }

    environment {
        KATALON_PATH = "C:\\Katalon\\KRE\\katalonc.exe"
        PROJECT_PATH = "%WORKSPACE%\\kre-test.prj"
        REPORT_DIR = "%WORKSPACE%\\Reports"
    }

    stages {
        stage('Run Test Suites per Modul') {
            steps {
                withCredentials([string(credentialsId: 'KATALON_API_KEY', variable: 'API_KEY')]) {
                    script {
                        // List folder modul di Test Suites
                        def modulDirs = []
                        bat(script: "dir /b /ad \"%WORKSPACE%\\Test Suites\"", returnStdout: true)
                            .trim()
                            .split("\r\n")
                            .each { modul ->
                                modulDirs << modul
                            }

                        echo "Found modules: ${modulDirs}"

                        modulDirs.each { modul ->
                            echo "Running module: ${modul}"

                            // List semua test suite di modul
                            def tsList = []
                            bat(script: "dir /b /s \"%WORKSPACE%\\Test Suites\\${modul}\\*.ts\"", returnStdout: true)
                                .trim()
                                .split("\r\n")
                                .each { filePath ->
                                    def relativePath = filePath.replace("${WORKSPACE}\\", "").replace("\\", "/")
                                    tsList << relativePath
                                }

                            tsList.each { ts ->
                                echo "Running Test Suite: ${ts}"
                                def modulReportDir = "${REPORT_DIR}\\${modul}"
                                bat "if not exist \"${modulReportDir}\" mkdir \"${modulReportDir}\""

                                bat """
                                %KATALON_PATH% -noSplash -runMode=console ^
                                -projectPath="${PROJECT_PATH}" ^
                                -retry=0 ^
                                -testSuitePath="${ts}" ^
                                -browserType="Chrome" ^
                                -executionProfile="default" ^
                                -apiKey=%API_KEY% ^
                                -reportFolder="${modulReportDir}" ^
                                --config -proxy.auth.option=NO_PROXY -proxy.system.option=NO_PROXY -proxy.system.applyToDesiredCapabilities=true ^
                                -webui.autoUpdateDrivers=true -studioAssist.provider="katalon_ai"
                                """
                            }
                        }
                    }
                }
            }
        }

        stage('Publish Reports') {
            steps {
                publishHTML(target: [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'Reports',
                    reportFiles: '**/index.html',
                    reportName: 'Katalon Test Report'
                ])
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Reports saved in ${REPORT_DIR}"
        }
    }
}