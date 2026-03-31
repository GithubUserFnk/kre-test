pipeline {
    agent { label 'Windows' }

    environment {
        KATALON_PATH = "C:\\Katalon\\KRE\\katalonc.exe"
        PROJECT_PATH = "${WORKSPACE}\\kre-test.prj"
        REPORT_DIR = "${WORKSPACE}\\Reports"
    }

    stages {

        stage('Clean Reports') {
            steps {
                echo "Cleaning old reports..."
                bat "if exist \"${REPORT_DIR}\" rmdir /s /q \"${REPORT_DIR}\""
            }
        }

        stage('Run Test Suites per Modul') {
            steps {
                withCredentials([string(credentialsId: 'KATALON_API_KEY', variable: 'API_KEY')]) {
                    script {

                        def testSuitesDir = new File("${WORKSPACE}\\Test Suites")
                        if (!testSuitesDir.exists()) {
                            error "Folder Test Suites tidak ditemukan!"
                        }

                        def modulDirs = testSuitesDir.listFiles()
                            .findAll { it.isDirectory() }*.name
                        echo "Found modules: ${modulDirs}"

                        modulDirs.each { modul ->
                            def modulDir = new File(testSuitesDir, modul)
                            def tsFiles = modulDir.listFiles()
                                .findAll { it.name.endsWith('.ts') }*.name
                            echo "Running module: ${modul} -> ${tsFiles}"

                            tsFiles.each { tsFile ->
                                def tsPath = "Test Suites/${modul}/${tsFile.replace('.ts','')}"
                                def modulReportDir = new File(REPORT_DIR)
                                modulReportDir.mkdirs()

                                bat """
                                "${KATALON_PATH}" -noSplash -runMode=console ^
                                -projectPath="${PROJECT_PATH}" ^
                                -retry=0 ^
                                -testSuitePath="${tsPath}" ^
                                -browserType="Chrome" ^
                                -executionProfile="default" ^
                                -apiKey=%API_KEY% ^
                                -reportFolder="${modulReportDir.absolutePath}" ^
                                --config -proxy.auth.option=NO_PROXY -proxy.system.option=NO_PROXY -proxy.system.applyToDesiredCapabilities=true ^
                                -webui.autoUpdateDrivers=true -studioAssist.provider="katalon_ai"
                                """
                            }
                        }
                    }
                }
            }
        }

        stage('Clean Root Report Files') {
            steps {
                // hapus file liar selain folder timestamp
                bat """
                cd "${REPORT_DIR}"
                for %%f in (*) do (
                    if not exist "%%f\\" (
                        del /f /q "%%f"
                    )
                )
                """
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
            echo "Reports saved in ${REPORT_DIR}"
        }
    }
}