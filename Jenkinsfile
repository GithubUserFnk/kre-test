pipeline {
    agent { label 'windows' }

    environment {
        KATALON_PATH = "C:\\katalon\\kre\\katalonc.exe"
        PROJECT_PATH = "%WORKSPACE%\\kre-test.prj"
        API_KEY = "2ef90766-f658-4064-be9d-0d19c8f703d3"
    }

    stages {
        stage('Run All Test Suites in Test Suites Folder') {
            steps {
                script {
                    // Path folder Test Suites
                    def tsFolder = "${WORKSPACE}\\Test Suites"

                    // List untuk semua test suite
                    def tsList = []

                    // Scan semua file .ts (test suite) di folder dan subfolder
                    new File(tsFolder).eachFileRecurse { file ->
                        if(file.name.endsWith(".ts")) {
                            // Buat relative path seperti KRE mau
                            def relativePath = file.path.replace("${WORKSPACE}\\", "").replace("\\", "/")
                            tsList << relativePath
                        }
                    }

                    echo "Found Test Suites: ${tsList}"

                    // Loop dan run tiap test suite
                    tsList.each { ts ->
                        echo "Running Test Suite: ${ts}"
                        bat """
                        %KATALON_PATH% -noSplash -runMode=console ^
                        -projectPath="${PROJECT_PATH}" ^
                        -retry=0 ^
                        -testSuitePath="${ts}" ^
                        -browserType="Chrome" ^
                        -executionProfile="default" ^
                        -apiKey=${API_KEY} ^
                        --config -proxy.auth.option=NO_PROXY -proxy.system.option=NO_PROXY -proxy.system.applyToDesiredCapabilities=true ^
                        -webui.autoUpdateDrivers=true -studioAssist.provider="katalon_ai"
                        """
                    }
                }
            }
        }
    }
}