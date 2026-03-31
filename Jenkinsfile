pipeline {
    agent { label 'Windows' }

    environment {
        KATALON_PATH = "C:\\Katalon\\KRE\\katalonc.exe"
        PROJECT_PATH = "${env.WORKSPACE}\\kre-test.prj"
        REPORT_DIR = "${env.WORKSPACE}\\Reports"
    }

    stages {
        stage('Run Test Suites per Modul') {
            steps {
                withCredentials([string(credentialsId: 'KATALON_API_KEY', variable: 'API_KEY')]) {
                    script {
                        // List modul (folder) di Test Suites
                        def modulDirs = bat(
                            script: "dir /b /ad \"${env.WORKSPACE}\\Test Suites\"",
                            returnStdout: true
                        ).trim().split("\r\n")

                        echo "Found modules: ${modulDirs}"

                        modulDirs.each { modul ->
                            echo "Running module: ${modul}"

                            // List semua .ts di modul
                            def tsListRaw = bat(
                                script: "dir /b \"${env.WORKSPACE}\\Test Suites\\${modul}\\*.ts\"",
                                returnStdout: true
                            ).trim()

                            if (tsListRaw) {
                                def tsList = tsListRaw.split("\r\n")
                                tsList.each { tsFile ->
                                    def tsName = tsFile.replace('.ts', '')
                                    def tsPath = "Test Suites/${modul}/${tsName}"
                                    echo "Running Test Suite: ${tsPath}"

                                    // Buat folder report per modul
                                    def modulReportDir = "${env.REPORT_DIR}\\${modul}"
                                    bat "if not exist \"${modulReportDir}\" mkdir \"${modulReportDir}\""

                                    // Jalankan Katalon CLI
                                    bat """
                                    %KATALON_PATH% -noSplash -runMode=console ^
                                    -projectPath="${PROJECT_PATH}" ^
                                    -retry=0 ^
                                    -testSuitePath="${tsPath}" ^
                                    -browserType="Chrome" ^
                                    -executionProfile="default" ^
                                    -apiKey=%API_KEY% ^
                                    -reportFolder="${modulReportDir}" ^
                                    --config -proxy.auth.option=NO_PROXY -proxy.system.option=NO_PROXY -proxy.system.applyToDesiredCapabilities=true ^
                                    -webui.autoUpdateDrivers=true -studioAssist.provider="katalon_ai"
                                    """
                                }
                            } else {
                                echo "No Test Suites found in module ${modul}"
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
            echo "Pipeline finished. Reports saved in ${env.REPORT_DIR}"
        }
    }
}