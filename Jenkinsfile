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
                        // Ambil list modul di Test Suites
                        def modulDirsRaw = bat(
                            script: "for /d %i in (\"${WORKSPACE}\\Test Suites\\*\") do @echo %~nxi",
                            returnStdout: true
                        ).trim()

                        def modulDirs = modulDirsRaw.split("\\r?\\n").findAll { it?.trim() }
                        echo "Found modules: ${modulDirs}"

                        modulDirs.each { modul ->
                            echo "Running module: ${modul}"

                            // Ambil semua TS (.ts) di modul
                            def tsRaw = bat(
                                script: "for %i in (\"${WORKSPACE}\\Test Suites\\${modul}\\*.ts\") do @echo %~nxi",
                                returnStdout: true
                            ).trim()

                            def tsList = tsRaw ? tsRaw.split("\\r?\\n").findAll { it?.trim() } : []

                            tsList.each { tsFile ->
                                def tsPath = "Test Suites/${modul}/${tsFile}".replace("\\", "/")
                                echo "Running Test Suite: ${tsPath}"

                                def modulReportDir = "${REPORT_DIR}\\${modul}"
                                bat "if not exist \"${modulReportDir}\" mkdir \"${modulReportDir}\""

                                bat """
                                ${KATALON_PATH} -noSplash -runMode=console ^
                                -projectPath="${PROJECT_PATH}" ^
                                -retry=0 ^
                                -testSuitePath="${tsPath}" ^
                                -browserType="Chrome" ^
                                -executionProfile="default" ^
                                -apiKey=${API_KEY} ^
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
            echo "Pipeline finished. Reports saved in ${env.REPORT_DIR}"
        }
    }
}