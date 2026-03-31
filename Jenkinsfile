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
                bat "if exist \"${REPORT_DIR}\" rmdir /s /q \"${REPORT_DIR}\""
            }
        }

        stage('Run Test Suites per Modul') {
            steps {
                withCredentials([string(credentialsId: 'KATALON_API_KEY', variable: 'API_KEY')]) {
                    script {

                        def testSuitesDir = new File("${WORKSPACE}\\Test Suites")

                        def modulDirs = testSuitesDir.listFiles()
                            .findAll { it.isDirectory() }
                            *.name

                        echo "Found modules: ${modulDirs}"

                        modulDirs.each { modul ->

                            def modulDir = new File(testSuitesDir, modul)

                            def tsFiles = modulDir.listFiles()
                                .findAll { it.name.endsWith('.ts') }
                                *.name

                            echo "Running module: ${modul} -> ${tsFiles}"

                            tsFiles.each { tsFile ->

                                // 🔥 FIX DI SINI (hapus .ts)
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