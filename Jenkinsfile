pipeline {
    agent { label 'Windows' }

    environment {
        KATALON_PATH = "C:\\Katalon\\KRE\\katalonc.exe"
        PROJECT_PATH = "%WORKSPACE%\\kre-test.prj"
    }

    stages {
        stage('Run All Test Suites') {
            steps {
                withCredentials([string(credentialsId: 'KATALON_API_KEY', variable: 'API_KEY')]) {
                    script {
                        // Loop test suite seperti sebelumnya
                        def tsFolder = "${WORKSPACE}\\Test Suites"
                        def tsList = []
                        new File(tsFolder).eachFileRecurse { file ->
                            if(file.name.endsWith(".ts")) {
                                def relativePath = file.path.replace("${WORKSPACE}\\", "").replace("\\", "/")
                                tsList << relativePath
                            }
                        }

                        tsList.each { ts ->
                            bat """
                            %KATALON_PATH% -noSplash -runMode=console ^
                            -projectPath="${PROJECT_PATH}" ^
                            -retry=0 ^
                            -testSuitePath="${ts}" ^
                            -browserType="Chrome" ^
                            -executionProfile="default" ^
                            -apiKey=%API_KEY% ^
                            --config -proxy.auth.option=NO_PROXY -proxy.system.option=NO_PROXY -proxy.system.applyToDesiredCapabilities=true ^
                            -webui.autoUpdateDrivers=true -studioAssist.provider="katalon_ai"
                            """
                        }
                    }
                }
            }
        }
    }
}