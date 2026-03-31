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
                        // --- List folder modul di Test Suites ---
                        def modulDirs = []
                        def rawModules = bat(
                            script: "dir /b /ad \"%WORKSPACE%\\Test Suites\"",
                            returnStdout: true
                        ).trim().split("\r\n")

                        rawModules.each { modul ->
                            if(modul?.trim()) {
                                modulDirs << modul.trim()
                            }
                        }

                        echo "Found modules: ${modulDirs}"

                        modulDirs.each { modul ->
                            echo "Running module: ${modul}"

                            // --- List semua test suite di modul ---
                            def tsList = []
                            def rawTS = bat(
                                script: "dir /b /s \"%WORKSPACE%\\Test Suites\\${modul}\\*.ts\"",
                                returnStdout: true
                            ).trim().split("\r\n")

                            rawTS.each { tsPath ->
                                if(tsPath?.trim()) {
                                    def relativePath = tsPath.replace("${WORKSPACE}\\", "").replace("\\", "/")
                                    tsList << relativePath
                                }
                            }

                            tsList.each { ts ->
                                echo "Running Test Suite: ${ts}"
                                def modulReportDir = "${env.REPORT_DIR}\\${modul}"
                                bat "if not exist \"${modulReportDir}\" mkdir \"${modulReportDir}\""

                                // --- Jalankan Katalon CLI ---
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
                // --- Publish HTML report dari semua modul ---
                publishHTML(target: [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: "${env.REPORT_DIR}",
                    reportFiles: '**/index.html',
                    reportName: 'Katalon Test Report'
                ])
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Reports saved in ${env.REPORT_DIR}"
        }
    }
}