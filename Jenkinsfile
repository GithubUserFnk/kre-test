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
                        def testSuitesDir = new File("${WORKSPACE}\\Test Suites")
                        def modulDirs = []

                        // List modul (folder)
                        testSuitesDir.eachDir { dir ->
                            modulDirs << dir.name
                        }

                        echo "Found modules: ${modulDirs}"

                        modulDirs.each { modul ->
                            echo "Running module: ${modul}"

                            def modulDir = new File(testSuitesDir, modul)
                            def tsList = []

                            // List semua .ts di modul
                            modulDir.eachFileRecurse { file ->
                                if(file.name.endsWith(".ts")) {
                                    def relativePath = file.path.replace("${WORKSPACE}\\", "").replace("\\", "/")
                                    tsList << relativePath
                                }
                            }

                            tsList.each { ts ->
                                echo "Running Test Suite: ${ts}"
                                def modulReportDir = "${env.REPORT_DIR}\\${modul}"
                                modulReportDir = modulReportDir.replace("/", "\\") // Windows safe
                                bat "if not exist \"${modulReportDir}\" mkdir \"${modulReportDir}\""

                                // Jalankan Katalon CLI
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
                    reportDir: "${env.WORKSPACE}\\Reports",
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