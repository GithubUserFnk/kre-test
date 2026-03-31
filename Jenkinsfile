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

                        // ambil folder modul
                        def modulDirs = testSuitesDir.listFiles()
                            .findAll { it.isDirectory() }
                            *.name

                        echo "Found modules: ${modulDirs}"

                        modulDirs.each { modul ->
                            echo "Running module: ${modul}"

                            def modulDir = new File(testSuitesDir, modul)

                            // ambil semua file .ts di dalam modul
                            def tsFiles = modulDir.listFiles()
                                .findAll { it.name.endsWith('.ts') }
                                *.name

                            echo "Found TS: ${tsFiles}"

                            tsFiles.each { tsFile ->

                                // remove .ts (WAJIB buat Katalon CLI)
                                def tsPath = "Test Suites/${modul}/${tsFile.replaceFirst(/\\.ts$/, '')}"
                                    .replace("\\", "/")

                                echo "Running Test Suite: ${tsPath}"

                                // buat folder report per modul
                                def modulReportDir = new File(REPORT_DIR, modul)
                                if (!modulReportDir.exists()) {
                                    modulReportDir.mkdirs()
                                }

                                // execute Katalon
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
                echo "Publishing HTML Reports..."
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