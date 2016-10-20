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

stage('Update ES Health') {
  def esHost = getEsHost()
  node('docker') {
    def url = "${esHost}/_cluster/health?pretty=true"
    def resp = httpRequest acceptType: 'APPLICATION_JSON', httpMode: 'GET', authentication: "es-auth", url: "$url", validResponseCodes: '100:500'
    def respObj = jsonParse(resp.content)
    println "es resp: ${resp.content}"
    
    //set up path for index
    def dateSuffix = new Date().format( 'yyyy-MM' )
    def esIndex = "es-health-$dateSuffix"
    def indexUrl = "${esHost}/${esIndex}/health/"
    def indexResp = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'PUT', requestBody: "$resp.content", authentication: "es-auth", url: "$url", validResponseCodes: '100:500'
    def indeRrespObj = jsonParse(indexResp.content)
    println "es health index resp: ${indeRrespObj}"
  }
}
