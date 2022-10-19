def environment () {
    builds =  [:]
    tests  =  [:]
    run    =  ['build': [], 'test':[]]
    dotnet = [
        'standard': ['1.3', '2.0'],
        'src':      findFiles(glob: 'src/Serilog.Sinks.File/*.csproj') as List,
        'tests':    findFiles(glob: 'test/Serilog.Sinks.File.Tests/*.csproj') as List
    ]
}

def build_src (standard, project) {
    sh """
       dotnet build -f netstandard${standard} -c Release ${project} -o netstandard${standard}
       """
}

def test_build (test) {
    sh """
       dotnet test -f netcoreapp2.0  -c Release ${test}
       """
}

node {
    stage ('init env') {
        cleanWs()
        checkout scm: ([
                    $class: 'GitSCM',
                    userRemoteConfigs: [[url: 'https://github.com/serilog/serilog-sinks-file.git']],
                    branches: [[name: 'main']]
            ])
        environment ()
    }
    stage ('compile builds') {
        dotnet.standard.each { standard ->
            for (file in dotnet.src) {
                run.build.add(['standard': standard, 'project': file.path])
            }
        }
    }
    stage ('compile tests') {
        for (file in dotnet.tests) {
            run.test.add(['test': file.path])
        }
    }
    stage ('compile parallel') {
        run.build.each { build -> 
            builds[build] = {  -> build_src (build.standard, build.project) }
        }
        run.test.each { test -> 
            tests[test] = {  -> test_build (test.test) }
        }
    }
    stage ('run parallel') {
        docker.image("mcr.microsoft.com/dotnet/sdk:6.0-alpine").inside {
            parallel builds
            parallel tests
        }
    }
    stage ('archive artifacts') {
        archiveArtifacts artifacts: 'netstandard*/*', onlyIfSuccessful: true
    }
}