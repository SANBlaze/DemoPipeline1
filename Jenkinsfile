
import groovy.transform.Field

@Library('SANBlazeAPI') _

@Field String default_ipaddr = ""

node {
    script {
        if (fileExists('param_ipaddr.txt')){
            default_ipaddr = readFile file: 'param_ipaddr.txt'
            echo "param_ipaddr.txt exists with value=" + default_ipaddr
        } else {
            echo "param_ipaddr.txt does not exist"
        }
//        echo "param ipaddr=" + params.ipaddr
//        echo "default_ipaddr=" + default_ipaddr
    }
}

pipeline {
//    environment {
//        savedipaddr = "2.1.1.1"
//   }
    agent any
    parameters {
//      booleanParam(defaultValue: true, description: '', name: 'userFlag')
        string(defaultValue: "${default_ipaddr}", description: 'SANBlaze system ip(v4) address', name: 'ipaddr')
        choice(
            name: 'choiceVar',
            choices:"oldchoice\nnewchoice",
            description: "Example of a choice param"
        )
    }
  //  def foo = "bar"
  //  options {
  //     paramList = [
  //      [$class: 'StringParameterValue', name: 'repo_user', value: "${repo_user}"],
  //  ]
  //  }
    stages {
       stage('Load Params') {
            steps {
                echo "Using params.ipaddr=" + params.ipaddr
                script {
                    if (params.ipaddr == ""){
                        echo "IP Address is not set!"
                        currentBuild.result = 'ABORTED'
                        error('ipaddr not set.  Restart job and enter ip address')
                    } else {
                        echo "Writing to param_ipaddr.txt ipaddr=" + params.ipaddr
                        writeFile file: 'param_ipaddr.txt', text: params.ipaddr
                    }
                }
            }
        }      
       stage('Delete Tests') {
            steps {
                script {
                    sanblaze.deleteAllTests("192.168.100.103", 2, 0, 200, 1)
                }
            }
        }      
        stage('Stage Tests') {
            steps {
                script {
//                  sanblaze.sayGreeting("Hello", "If you see Hello, library is working")
                    sanblaze.stageTest("192.168.100.103", "2", "0", "200", "1", "-1", "/IO_Tests/Read_Seq_64thr_1blk.sh", "1", "10")
                    sanblaze.stageTest("192.168.100.103", "2", "0", "200", "1", "-1", "/IO_Tests/Read_Seq_64thr_32blk.sh", "1", "10")
                    sanblaze.stageTest("192.168.100.103", "2", "0", "200", "1", "-1", "/IO_Tests/Read_Seq_64thr_64blk.sh", "1", "10")
                    // next test will fail
//                    sanblaze.stageTest("192.168.100.103", "2", "0", "200", "1", "-1", "/NVMe_Generic/NVMe_IdentifyAllocatedNSIDs.sh", "1", "0")
                    // next test will get skipped
                    sanblaze.stageTest("192.168.100.103", "2", "0", "200", "1", "-1", "/NVMe_Generic/NVMe_IdentifyNSIdentDescript.sh", "1", "0")
                    // next test will end in warning
//                    sanblaze.stageTest("192.168.100.103", "2", "0", "200", "1", "-1", "/NVMe_MI_Basic/NVMe_MI_Basic_Status.sh", "1", "0")
                    sanblaze.stageTest("192.168.100.103", "2", "0", "200", "1", "-1", "/NVMe_Resets/NVMe_Controller_Reset.sh", "1", "0")
                    sanblaze.stageTest("192.168.100.103", "2", "0", "200", "1", "-1", "/IO_Tests/R25W75_Seq_64thr_16blk.sh", "1", "0")
                }
            }
        }
       stage('Run Tests') {
            steps {
                script {
                    /*
                     * This will ask the user for input (mouse over the paused stage)
                     */
//                  sanblaze.sayGreeting("Hello", "If you see Hello, library is working")
                    sanblaze.scriptAction("192.168.100.103", 2, 0, 200, 1, "", "clear", "start")
                    sanblaze.scriptAction("192.168.100.103", 2, 0, 200, 1, 1, "start", "start")
                    sanblaze.waitForState("192.168.100.103", 2, 0, 200, 1, 1, "Running", 5)
                    sanblaze.waitForState("192.168.100.103", 2, 0, 200, 1, 6, "Passed", 90)
                    echo "Done with waitForState"
                   def logfile
                    logfile = httpRequest "http://192.168.100.103/rest/sanblazes/2/ports/0/targets/200/luns/1/tests/1/Read_Seq_64thr_1blk.sh.log"
                    writeFile file: 'Read_Seq_64thr_1blk.sh.log', text: logfile.content
                    logfile = httpRequest "http://192.168.100.103/rest/sanblazes/2/ports/0/targets/200/luns/1/tests/2/Read_Seq_64thr_32blk.sh.log"
                    writeFile file: 'Read_Seq_64thr_32blk.sh.log', text: logfile.content
                    logfile = httpRequest "http://192.168.100.103/rest/sanblazes/2/ports/0/targets/200/luns/1/tests/3/Read_Seq_64thr_64blk.sh.log"
                    writeFile file: 'Read_Seq_64thr_64blk.sh.log', text: logfile.content
                }
            }
        }
        stage('Get Log Files') {
            steps {
                script {
                    archiveArtifacts '*.log'
                }
            }
        }
        stage('Get Results') {
            steps {
                script {
                    def response = httpRequest "http://192.168.100.103/goform/JsonApi?op=rest/sanblazes/2/ports/0/targets/200/luns/1/jUnit.xml"
                    writeFile file: 'response.txt', text: response.content
//                    println('Status: '+ response.status)
//                    println('Response: '+ response.content)
                }
            }
        }
    }
    post { 
        always {
            echo 'status=' + currentBuild.result
            echo 'I will always say Hello again!'
            junit 'response.txt'
        }
        unstable {
            echo "UNSTABLE runs after ALWAYS when test results were unstable"
        }
    failure {
            echo "FAILURE runs after ALWAYS when test results failed"
        }
    }
}

