pipeline {
    agent { label 'Windows' }

    environment {
        KATALON_PATH = "C:\\Katalon\\KRE\\katalonc.exe"
        PROJECT_PATH = "${WORKSPACE}\\kre-test.prj"
        REPORT_DIR = "${WORKSPACE}\\Reports"
    }

    stages {
        stage('Run Test Suites per Modul') {
            steps {
                withCredentials([string(credentialsId: 'KATALON_API_KEY', variable: 'API_KEY')]) {
                    script {
                        // 1. Ambil folder modul
                        def testSuitesDir = new File("${WORKSPACE}\\Test Suites")
                        if (!testSuitesDir.exists()) {
                            error "Folder Test Suites tidak ditemukan!"
                        }

                        def modulDirs = testSuitesDir.listFiles().findAll { it.isDirectory() }*.name
                        echo "Found modules: ${modulDirs}"

                        modulDirs.each { modul ->
                            echo "Running module: ${modul}"
                            def modulDir = new File(testSuitesDir, modul)

                            // 2. Ambil semua TS (.ts)
                            def tsFiles = modulDir.listFiles().findAll { it.name.endsWith('.ts') }*.name
                            echo "Found TS: ${tsFiles}"

                            tsFiles.each { tsFile ->
                                def tsPath = "Test Suites/${modul}/${tsFile}".replace("\\", "/")
                                echo "Running Test Suite: ${tsPath}"

                                // 3. Folder report per modul
                                def modulReportDir = new File(REPORT_DIR, modul)
                                if (!modulReportDir.exists()) {
                                    modulReportDir.mkdirs()
                                }

                                // 4. Jalankan Katalon CLI
                                bat """
                                "${KATALON_PATH}" -noSplash -runMode=console ^
                                -projectPath="${PROJECT_PATH}" ^
                                -retry=0 ^
                                -testSuitePath="${tsPath}" ^
                                -browserType="Chrome" ^
                                -executionProfile="default" ^
                                -apiKey=${API_KEY} ^
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
            echo "Pipeline finished. Reports saved in ${REPORT_DIR}"
        }
    }
}