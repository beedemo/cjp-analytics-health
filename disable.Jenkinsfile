#!groovy
import groovy.json.*

@NonCPS
def jsonParse(def json) {
    new groovy.json.JsonSlurperClassic().parseText(json)
}

@NonCPS
def getEsHost() {
    return System.getenv().ES_HOST   
}
//configure to run every 15 minutes and only keep last 5 builds
properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '5')), pipelineTriggers([cron('H/15 * * * *')])])
stage('Update ES Health') {
  def esHost = getEsHost()
  node('docker') {
    def url = "${esHost}/_cluster/health?pretty=true"
    def resp = httpRequest acceptType: 'APPLICATION_JSON', httpMode: 'GET', authentication: "es-auth", url: "$url", validResponseCodes: '100:500'
    def respObj = jsonParse(resp.content)
    println "es resp: ${resp.content}"
      
    def timestamp = new Date().format("EEE, d MMM yyyy HH:mm:ss Z")
    def healthJson = """
      {
        "cluster_name":"$respObj.cluster_name",
        "status":"$respObj.status",
        "number_of_nodes":$respObj.number_of_nodes,
        "number_of_data_nodes":$respObj.number_of_data_nodes,
        "@timestamp": "$timestamp"
      }
    """
    
    //set up path for index
    def dateSuffix = new Date().format( 'yyyy-MM' )
    def esIndex = "es-health-$dateSuffix"
    def indexUrl = "${esHost}/${esIndex}/health/"
    def indexResp = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'POST', requestBody: "$healthJson", authentication: "es-auth", url: "$indexUrl", validResponseCodes: '100:500'
    println "es health index resp: ${indexResp.content}"
  }
}
